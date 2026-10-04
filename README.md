# Marvelous

Marvelous is a UCI chess engine written in C++20. It pairs an alpha-beta search with an NNUE evaluation and runs in any UCI GUI or through [lichess-bot](https://github.com/lichess-bot-devs/lichess-bot).

## Building

You need a C++20 compiler (clang or gcc) and `make`.

```sh
make            # optimized build for this machine
make pgo        # profile-guided build, a few percent faster
```

The network in `net/marvelous.nnue` is embedded into the binary at build time, so the executable is self-contained.

On Windows, build from an MSYS2 MinGW64 shell (`pacman -S mingw-w64-x86_64-clang make`); the result is a static `marvelous.exe`.

## UCI options

| Option | Default | Notes |
| --- | --- | --- |
| Hash | 256 | Transposition table size in MB |
| Threads | 1 | Search threads (Lazy SMP) |
| Move Overhead | 30 | Milliseconds kept in reserve per move for communication lag |
| MultiPV | 1 | Number of principal variations to report |
| Contempt | 0 | Centipawns a draw is worth less than equality to the side to move at the root |
| Ponder | false | Accepted so GUIs can enable pondering |
| EvalFile | `<embedded>` | Load a different network file |
| UCI_ShowWDL | false | Accepted for compatibility |

`bench [depth]` prints a node count and speed over a fixed set of positions. `go perft <depth>` counts legal move paths.

## Evaluation

The network is `(768 x 16 king buckets -> 1024) x 2 -> 1 x 8 output buckets` with squared clipped ReLU, horizontally mirrored king buckets and output buckets chosen by piece count. It is updated incrementally with lazily computed accumulators and a refresh cache keyed by king bucket.

It was trained with [bullet](https://github.com/jw1912/bullet) on the `test80-2022-08-aug` training data published by the Stockfish project from Leela Chess Zero games (ODbL).

## Search

Principal variation search with aspiration windows, a clustered transposition table, iterative deepening and Lazy SMP. Pruning and reductions include reverse futility pruning, razoring, null move pruning with verification, ProbCut, late move reductions and pruning, futility and SEE pruning, and history pruning. Extensions are singular, double and triple, with multi-cut and negative extensions. Move ordering uses threat-aware butterfly history, capture history, continuation history and pawn history. Static evaluation is corrected by pawn, minor-piece, non-pawn and continuation correction histories. Upcoming repetitions are detected with cuckoo tables.

## Thanks

The search draws on ideas published in the source of Stockfish, Obsidian, Alexandria, PlentyChess, Berserk, Viridithas and Integral, and on the testing practice of the OpenBench and fishtest communities. Matches were run with [fastchess](https://github.com/Disservin/fastchess).

## License

MIT, see [LICENSE](LICENSE).
