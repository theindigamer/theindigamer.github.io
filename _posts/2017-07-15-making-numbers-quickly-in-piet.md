---
layout: post
title: "Making numbers quickly in Piet"
categories: [programming]
tags:
- piet
- math
- ocaml
---

[piet]: http://www.dangermouse.net/esoteric/piet.html
[tutorial]: http://homepages.vub.ac.be/~diddesen/piet/index.html
[rpn]: https://en.wikipedia.org/wiki/Reverse_polish_notation
[mseq]: https://math.stackexchange.com/questions/2355111/making-numbers-cheaply
[trie]: https://en.wikipedia.org/wiki/Trie
[results-12000]: https://github.com/theindigamer/theindigamer.github.io/blob/e2ce7643d87fcadd4fa90e333bb33cb41b83eca8/data/results-12000.txt

(Note: This article is based on my experience asking a [question][mseq] on the Math StackExchange and coming up with a solution based off one given in an answer. If you only care about the math, just read the original question's text and skip the first two sections.)

1, 2, 3, 4, 5, 8, 7, 11, 13, 22, 41, 44, 94, 176, 177, 472, 697, 1067, 1391, 3557, 8493, ...

### Background

[Piet][piet] is an esoteric programming language where programs take the form of drawings (bitmap images, to be more precise). Instead of describing how Piet works, let me just describe the relevant subset that we are interested in.

The program is broken up into "codels". A codel is basically a square arrangement of pixels (identically coloured) that represents one code unit for Piet. A codel may take one of 20 colours -- white, black and light, medium and dark variants of red, green, blue, cyan, magenta and yellow.

At any given point, the program is in some codel. A Piet interpreter would maintain a stack (containing numbers) for computations. The normal push and pop operations are supported for the stack. Calling an add operation would mean popping off the top two values on the stack, adding them and pushing the result back onto the stack. In postfix notation, this is represented like [5,6,3,4,+] → [5,6,7] where the top of the stack is on the right.

Colour blocks are contiguous (in 2D) sequences of codels. So colour blocks may have size 1 (in codels) or more.

Colour transitions dictate what operation ought to be done. For example, the transition light red → (medium) red means that the size of the previous (light red) colour block is pushed onto the stack. For example, the following pseudocode just pushes 3 onto the stack and exits:

```
lightred lightred lightred red.
```

(For simplicity, we look only at linear programs -- program flow is from left to right.)

The math operations support by Piet include {~,+,-,\*,/,%,@} where ~ is a not operation (0 maps to 1 and the rest map to zero), {+,-,\*,/} are ordinary addition, subtraction, multiplication and integer division (floored), % is the modulo operation and @ is a unary duplication operation. For example, [13,5,%] → [3] and [3,@,\*] → [3,3,\*] → [9].

(If you are still confused about what the heck is going on, then this [Piet tutorial][tutorial] might be helpful. Only the sections before "Controlling the flow" are relevant for understanding the current article. You may also find Wikipedia's page on [reverse polish notation][rpn] helpful.)

### Math problem

(This section is a super condensed version of the [question][mseq] posted on StackExchange.)

In the earlier example it is clear that pushing 3 onto the stack "costs" 3 transitions (marked with `|`).

```
lightred | lightred | lightred | red.
```

From the Piet specification, it is easy to deduce that the minimum cost of pushing a number _n_ directly onto the stack is exactly _n_ transitions. One can also deduce that applying any math operation costs 1 transition.

What about indirectly pushing numbers? For example (with implicit pushes), [3,@,@,+,+] ends up (effectively) pushing the number 9 onto the stack with cost 7. However [3,@,*] also ends up (effectively) doing the same thing but with cost 5.

So a natural question to ask is: given a number _n_ which we'd like to push onto the stack, what is the cheapest sequence of operations that (effectively) does the same thing?

### Stupid algorithm

Imagine an infinite [prefix tree][trie] which contains all possible legal paths. For example, [3,3,-] is a legal path whereas [3,+] isn't (there aren't two numbers to add). Since [3,3,-] is a legal path, the tree's root (empty) has a child node `3` which has a child node `3` which has a child node `-` which has child nodes `{@, 1, 2, 3, ...}` (no binary operations are children of `-` because that would make an illegal path).

Let us also put the corresponding (effective) sequence at the nodes, i.e., path [3,@,2,-] will have the sequences as {[3], [3,3], [3,3,2], [3,1]}. The [3,3,2] will be stored at the node with `2` and so on. While traversing this tree, if we see that that the sequence at our current node is of length 1, say [_m_], then the ancestors of the current node (including itself) describe a candidate cheapest path for creating _m_.

For simplicity, we also maintain the cumulative cost of ancestors (including itself) at the nodes.

We do a depth first search of this tree up to some fixed depth (i.e. an arbitrarily chosen maximum cost). While searching, we maintain a look up table mapping sequences to minimum cost, updating as necessary. If we arrive at a node where the the sequence has a saved mapping, we compare the costs -- if the new cost is equal to or higher than the saved cost then we stop going deeper into the tree. Similarly, if the current sequence is too long to shrink to length 1 (using operations) in the 'budget' (max cost minus current cost), then we stop going deeper.

(Yes, I know this algorithm is exponential; if you have a better solution then please get in touch :).)

Here is a Ocaml implementation of the algorithm described above:

```ocaml
open Batteries

module MakeCheap = struct
  module H = BatHashtbl
  module V = BatVect

  type seq = int list
  let int_max = BatInt.max_num
  type binary_op = PAdd | PSub | PMul | PDiv | PMod
  type elem = Number of int | PDup | Binary of binary_op
  type path = elem list

  let string_of_elem = function
    | Number x -> string_of_int x
    | PDup -> "@"
    | Binary PAdd -> "+"
    | Binary PSub -> "-"
    | Binary PMul -> "*"
    | Binary PDiv -> "/"
    | Binary PMod -> "%"

  type full_history = {
    maxc : int ref;
    best : (int, int * elem list) H.t; (* number -> (cost, path) *)
    hist : (seq, int) H.t;             (* sequence -> length of best path *)
    lim  : int;
    def  : elem list;
  }

  type path_history = {
    cost : int;
    path : path;
    st_l : seq list;
  }

  let rec branch path_h full_h =
      let make_kids acc = function
        | PDup -> (match path_h.st_l with
            | (h :: t) :: _ -> (PDup, path_h.cost + 1, h :: h :: t) :: acc
            | _ -> acc)
        | Binary x
          -> (match path_h.st_l with
              | (a :: b :: t) :: _ ->
                if (b < 0 && x = PMod) ||
                   (a = 0 && (x = PDiv || x = PMod)) then acc
                else let ab_op = (match x with
                    | PAdd -> b + a
                    | PSub -> b - a
                    | PMul -> b * a
                    | PDiv -> b / a
                    | PMod -> b mod a) in
                  (Binary x, path_h.cost + 1, ab_op :: t) :: acc
              | _ -> acc)
        | Number x -> (Number x, path_h.cost + x,
                       (match path_h.st_l with
                        | [] -> [x]
                        | h :: _ -> x :: h)) :: acc in
      let traverse path_h fh (el, cost, seq) =
        if cost > !(fh.maxc) then fh
        else
          let old_c = match H.find_option fh.hist seq with
            | Some x -> x
            | None -> int_max in
          if cost >= old_c then fh
          else
            let _ =
              H.replace fh.hist seq cost;
              if List.length seq = 1 then
                let x = List.hd seq in
                if x < 0 then () else
                  let old_c = match H.find_option fh.best x with
                    | Some (y, _) -> y
                    | None -> int_max in
                  if cost < old_c then begin
                    H.replace fh.best x (cost, el :: path_h.path);
                    if old_c = !(fh.maxc) then
                      fh.maxc :=
                        List.range 1 `To fh.lim
                        |> List.map (fst % H.find fh.best)
                        |> List.reduce BatInt.max
                  end
            in
            branch {cost = cost; path = el :: path_h.path;
                    st_l = seq :: path_h.st_l} fh
      in

      if (not (List.is_empty path_h.st_l) &&
         (BatInt.max 0 (List.length (List.hd path_h.st_l) - 1)) + path_h.cost
         >= !(full_h.maxc)) then full_h
      else
        full_h.def
        |> List.fold_left make_kids []
        |> List.fold_left (traverse path_h) full_h

  let find_all_cheaply max_num goal =
    let start_path_h = {cost = 0; path = []; st_l = [];} in
    let start_full_h guess = {
      best = (List.range 1 `To goal)
             |> List.map (fun x -> (x, (guess, [])))
             |> H.of_list;
      maxc = ref guess;
      hist = H.create 1024;
      def = [PDup;]
            |> List.append
               % List.map (fun x -> Binary x) @@ [PAdd; PSub; PMul; PDiv; PMod;]
            |> List.append
               % List.map (fun x -> Number x) @@ (List.range 1 `To max_num);
      lim = goal;
    } in

    let rec run_branch guess full_h =
      let fh = branch start_path_h full_h in
      (* retry if we didn't find a path for all numbers from 1 to goal *)
      if List.range 1 `To goal
         |> List.map ((=) guess % fst % H.find fh.best)
         |> List.reduce (||) then
        run_branch (guess + 2) (start_full_h (guess + 2))
      else fh
    in

    let full_h = run_branch 16 (start_full_h 16) in
    List.range 1 `To goal
    |> List.map (fun a -> (a, H.find full_h.best a))
    |> List.map (fun (a, (l,s)) -> (a,l, List.map (string_of_elem) @@ List.rev s))
    |> List.sort (fun (a1,_,_) (a2,_,_) -> compare a1 a2)
    |> List.map
      (fun (a,l,s) -> let _ = print_endline @@ Printf.sprintf "%5d %2d %s" a l
                        @@ List.fold_left (^) "" s in (a,l,s))

end

let _ = MakeCheap.find_all_cheaply 5 256
```

(I've forbidden the code from using modulo in case of negative divisors for reasons which I don't want to go in here.)

### Interesting (?) results

The full results for 1 to 12,000 are available [here][results-12000]. The first column is the number, the second column is the lowest cost possible and the last column is one possible lowest cost path of operations to construct the number.

The sequence at the start of the article represents the smallest numbers for a given cost. I've put it in a cost -> number format for convenience below.

1 -> 1, 2 -> 2, 3 -> 3, 4 -> 4, 5 -> 5, 6 -> 8, 7 -> 7, 8 -> 11, 9 -> 13, 10 -> 22, 11 -> 41, 12 -> 44, 13 -> 94, 14 -> 176, 15 -> 177, 16 -> 472, 17 -> 697, 18 -> 1067, 19 -> 1391, 20 -> 3557, 21 -> 8493, ...

(Also, I have the suspicion that the first cost 22 number is between 12,000 and 16,000 given that my laptop ran out of swap space for 16,000 :neutral_face:.)

This data analyis is easily done using Python:

```python
import pandas as pd
dfo = pd.read_csv("results-10000.txt",
        delim_whitespace = True,
        names = ["num", "cost", "seq"],
        header = None)
df = dfo.sort_values(by = ["cost", "num"])
grouped = df.groupby("cost")
print(grouped.first())
```

A scatter plot of cost vs number is shown below:

![Plot for cost vs number](http://i.imgur.com/AprHYSI.png)

Some other things I found cool (although I concede some are related to the implicit ordering of operations @, +, -, \*, /, % in my code):

* The first number to use division is 187 in [5,@,@,\*,\*,@,2,/,+]. Out of 12,000 paths, 899 use one or more division operations (there are 910 divisions in total).
* Exactly one number uses a modulo operation -- it is 9362 with the path [4,@,\*,@,\*,@,\*,@,3,@,\*,%,/] of cost 18. This generates 65536, duplicates it, does mod 9 (giving 7) and does 65536/7 = 9362 (floored). Wow.

(Python code for checking substrings: `dfo[dfo["seq"].str.contains('/')]`.)

### Get in touch!

If you have any questions or comments, please [open an issue](https://github.com/theindigamer/theindigamer.github.io/issues) on this blog's Github repo or send me an email: `theindigamer15` (gmail). :smile:
