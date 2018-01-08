# PorterDuff

PorterDuffXferMode

| Mode     | Alpha                           | Color                                                                   |
| -------- | ------------------------------- | ----------------------------------------------------------------------- |
| CLEAR    | 0                               | 0                                                                       |
| SRC      | As                              | Cs                                                                      |
| DST      | Ad                              | Cd                                                                      |
| SRC_OVER | As + (1 - As) \* Ad             | Cs + (1 - As) \* Ad                                                     |
| DST_OVER | Ad + (1 - Ad) \* As             | Cd + (1 - Ad) \* Cs                                                     |
| SRC_IN   | As \* Ad                        | Cs \* Ad                                                                |
| DST_IN   | As \* Ad                        | Cd \* As                                                                |
| SRC_OUT  | (1 - Ad) \* As                  | (1 - Ad) \* Cs                                                          |
| DST_OUT  | (1 - As) \* Ad                  | (1 - As) \* Cd                                                          |
| SRC_ATOP | Ad                              | Ad \* Cs + (1 - As) \* Cd                                               |
| DST_ATOP | As                              | As \* Cd + (1 - Ad) \* Cs                                               |
| XOR      | (1 - Ad) \* As + (1 - As) \* Ad | (1 - Ad) \* Cs + (1 - As) \* Cd                                         |
| DARKEN   | As + Ad - As \* Ad              | (1 - Ad) \* Cs + (1 - As) \* Cd + min(Cs, Cd)                           |
| LIGHTEN  | As + Ad - As \* Ad              | (1 - Ad) \* Cs + (1 - As) \* Cd + max(Cs, Cd)                           |
| MULTIPLY | As \* Ad                        | Cs \* Cd                                                                |
| SCREEN   | As + Ad - As \* Ad              | Cs + Cd - Cs \* Cd                                                      |
| ADD      | max(0, min(As + Ad, 1))         | max(0, min(Cs + Cd, 1))                                                 |
| OVERLAY  | As + Ad - As \* Ad              | if (2 \* Cd < Ad) {2 \* Cs \* Cd} else {As \* Ad - 2(Ad - Cs)(As - Cd)} |