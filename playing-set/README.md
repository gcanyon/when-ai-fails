# Let Them Play Set!

## tldr; large language models struggle with playing Set, but newer reasoning models like DeepThink-R1 and o3-mini are able to solve it reliably.

Set is a card game where players have to identify sets of three cards from a layout of 12. Each card features a combination of four attributes: shape, color, number, and shading. A valid set consists of three cards where each attribute is either the same on all three cards or different on each. The goal is to find such sets quickly and accurately.

Though this game is a solved computer problem — easily tackled by algorithms or deep learning — I thought it would be interesting to see if Large Language Models (LLMs) could figure it out.

Here's the prompt:

```
Set is a card game where 12 cards are laid out face up on the table for all of the players to see, and the players compete to find valid Sets of cards as quickly as they can, faster than the other players.

In the game Set, each card has a number of figures on it, all alike. The figures on a given card have four attributes, each with three possible values:

- Count: the number of figures on the card. The three possible values are: 1, 2, or 3.
- Shading: whether the figures are filled or not. The three possible values are: F (filled), E (empty), H (hashed)
- Color: the color of the figures. The three possible values are: R (red), G (green), P (purple)
- Shape: the shape of the figures. The three possible values are: D (diamond), O (oval), S (squiggle)

Each card can be described using the format: [Count][Color][Shading][Shape]

- Count: 1, 2, or 3 (number of shapes on the card)
- Shading: F (filled), E (empty), H (hashed)
- Color: R (red), G (green), P (purple)
- Shape: D (diamond), O (oval), S (squiggle)

In the game Set, a "valid Set" consists of exactly three cards, where, for each of the four figure attributes, the three cards either have the same value for the attribute, or all different values for the atribute. Put another way, for each attribute there CANNOT be two cards with one value, and another card with a different value. The order of the cards is irrelevant.

Considering just the Count: The count would be valid if it were all different, meaning 1, 2, and 3. It would also be valid if it were all the same: either 1, 1, and 1; or 2, 2, and 2; or 3, 3, and 3. The count would be INVALID if it were 2, 2, and 3; or if it were 3, 1, and 3; or if it were 2, 1, and 1. In all those cases there were two of one count, and one of another. There are other examples of that.

The same applies to the other three attributes: all Red is valid, Red, Green, and Purple is valid, all Green is valid, and all Purple is valid. But Red, Green, and Red is invalid, because there are two Reds and one Green.

So for example, these three cards make a valid Set:

- 1FPD (One Filled Purple Diamond)
- 2FPD (Two Filled Purple Diamond)
- 3FPD (Three Filled Purple Diamond)

They make a set because:
1. The Counts are all different: 1, 2, and 3
2. The Shading is all the same: Filled
3. The Color is all the same: Purple
4. The Shape is all the same: Diamond

These three cards do not make a valid Set:

- 2FPO (Two Filled Purple Oval)
- 2FPD (Two Filled Purple Diamond)
- 3FPS (Three Filled Purple Squiggle)

Although they all have the same Shading and Color, which qualifies, and they all have different Shapes, which qualifies, they have 2, 2, and 3 figures, which is not allowed. 

These five cards make two different valid Sets:

- 1FPO (One Filled Purple Oval)
- 2HRD (Two Filled Purple Diamond)
- 2ERS (Two Empty Red Squiggle)
- 3HGD (Three Hashed Green Diamond)
- 3EGS (Three Filled Purple Squiggle)

The two valid Sets are:

- 1FPO (One Filled Purple Oval)
- 2ERS (Two Empty Red Squiggle)
- 3HGD (Three Hashed Green Diamond)

These make a valid Set because:
1. The Counts are all different: 1, 2, and 3
2. The Shadings are all different: Filled, Empty, Hashed
3. The Colors are all different: Purple, Red, Green
4. The Shapes are all different: Oval, Squiggle, Diamond

and

- 1FPO (One Filled Purple Oval)
- 2HRD (Two Filled Purple Diamond)
- 3EGS (Three Filled Purple Squiggle)

These make a valid Set because:
1. The Counts are all different: 1, 2, and 3
2. The Shading is all the same: Filled
3. The Color is all the same: Purple
4. The Shapes are all different: Oval, Diamond, and Squiggle

Note how cards (in this case, one card) can be part of more than one valid Set.

You are given twelve Set cards, each represented by the notation defined above. The :

1. 2FRO (Two Filled Red Oval)
2. 1HPO (One Hashed Purple Oval)
3. 2FGO (Two Filled Green Oval)
4. 1EGS (One Empty Green Squiggle)
5. 3HGD (Three Hashed Green Diamond)
6. 2FRS (Two Filled Red Squiggle)
7. 2EGD (Two Empty Green Diamond)
8. 2EGS (Two Empty Green Squiggle)
9. 3HRD (Three Hashed Red Diamond)
10. 1EPO (One Empty Purple Oval)
11. 2HRO (Two Hashed Red Oval)
12. 3FGO (Three Filled Green Oval)

Your task is to identify and list all possible Sets from this list. 

Provide the sets in terms of their card numbers (e.g., Card 1, Card 2, Card 3).
```

## Results

| Model                                          | Finds Sets | Comments                                                                                                                                                                        |
| ---------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`GPT-4o`](./gpt-4o/answer.md)                 | ❌         | Suggests invalid sets. After multiple verification requests, it wrongly states that there are no sets.                                                                          |
| [`Sonnet-3.5`](./claude-sonnet-3.5//answer.md) | ❌         | At first attempt a clearly invalid set. After that it keeps failing, but it's honest about it.                                                                                  |
| [`Mistral`](./mistral/answer.md)               | ❌         | At first Mistral uses Python to figure out the set ([correctly](./mistral/answer-python.md)!). While smart, I feel this defeats the purpose of testing if the model can reason. |
| [`o3-mini`](./o3-mini/answer.md)               | ✅         | Correctly finds 3 sets after 1m 12s of thinking.                                                                                                                                |
| [`DeepThink-R1`](./deepthink-r1/answer.md)     | ✅         | Correclty finds 3 sets after 10 minutes of thinking.                                                                                                                            |

## About Set

Here’s what the game looks like:

![Example deck](./example-deck.jpeg)

There’s a shorthand notation for describing the attributes of a card. For instance, the card in the top left corner of the image above can be described as `O2RS`, meaning it has two solid red ovals.

Here’s the full definition of the notation:

```
[Shape][Number][Color][Shading]

Shape: O (oval), S (squiggle), D (diamond)
Number: 1, 2, or 3 (count of shapes)
Color: R (red), G (green), P (purple)
Shading: S (solid), O (outline), H (hashed/shaded)
```

Using this notation, any deck can be described in text. In this particular layout, there are three Sets:

1. Cards 3, 4, 5
2. Cards 5, 6, 10
3. Cards 10, 11, 12

My expectation was that the LLM will either be able to identify the sets or at least admit when it cannot find them. Failure I would define as suggesting invalid sets or claiming there are no sets when there are.
