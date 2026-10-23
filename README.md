# Web3 Reward Algo

A single Python function, `calculate_reward`, written as an answer to a
coding-task question ("Task 2: Reward Distribution, Q3"). It splits a
fixed stream of USDT rewards between users of a lending protocol in
proportion to the shares each user holds over a one-hour window.

There is no blockchain code here. "Web3" refers to the scenario in the
task, not to anything the code connects to.

## Contents

| File | What it is |
| --- | --- |
| `index.py` | `calculate_reward(events)` plus four example calls that print their results |
| `sweep.yaml`, `.github/ISSUE_TEMPLATE/sweep-template.yml` | Config for the Sweep AI bot, added by a bot PR; not used by the code |
| `LICENSE` | GNU General Public License v3.0 |

## How it works

`calculate_reward` takes a list of `(user, timestamp, share_adjust)`
tuples and returns a dict of `{user: reward}`:

1. Events are sorted by timestamp, and an end marker is added at t=3600.
2. For each interval between events, `2.778 * interval` USDT is split
   between users in proportion to their current shares.
3. The event's share change is then applied to that user.
4. Totals are rounded to 3 decimal places.

## Run it

Needs Python 3.9 or newer (it uses `list[...]` and `dict[...]` type hints).
There are no dependencies.

```bash
python3 index.py
```

Output:

```
Test 1: {'A': 5006.882, 'B': 4993.918}
Test 2: {'A': 10000.8}
Test 3: {'A': 41.67, 'B': 41.67}
Test 4: {}
```

To use the function from other code, note that importing `index` also
runs the four example prints.

## Status

Finished as a task submission and not maintained. Known gaps against the
task statement below:

- The rate is the rounded 2.778 USDT/sec, so a full hour pays out
  10,000.8 USDT rather than 10,000.
- The task's example returns `{'A': 5006.882, 'B': 4993.918}`, not the
  expected `{'A': 5005.556, 'B': 4994.444}`. Part of the gap is the
  rounded rate; the rest is the task itself: its worked example has B
  joining at t=1, while its event list has B joining at t=2. The code
  follows the event list.
- Shares are not clamped at zero. Removing more shares than a user holds
  gives that user negative shares.
- There is no automated test suite. A `unittest` file was added and later
  removed in the commit history; `index.py` only prints example results.

## The task

The question this code answers, from the original README (lightly
edited for formatting):

> Write a Python3 code that calculates how much each user should receive
> the reward. Please include any test cases to test the functionality,
> including normal and edge cases as well.
>
> Implement the Python function according to the format below:
>
> ```python
> def calculate_reward(events: list[tuple[str,int]]) -> dict[str, float]:
>     raise NotImplemented()
> ```
>
> Problem statement: We have 10,000 USDT to be distributed to our users
> who use our lending protocol during the timestamp 0 to 3600 (1 hour
> duration). The reward distribution algorithm is such that the 1000
> tokens are distributed at the rate of ~2.778 USDT / sec, where at each
> point in time, the tokens will be distributed proportionally based on
> the total number of user's shares.
>
> Each user's action can either be:
>
> 1. Increase shares by XX, or
> 2. Decrease shares by YY (user's shares can never be negative)
>
> Example: user action events, as a list of tuples of
> `(user, timestamp, share_adjust)`:
> `[("A", 0, 2), ("B", 2, 1), ("A", 10, -1)]`
>
> - User A increases 2 shares at time t=0
> - User B increases 1 share at time t=2
> - User A decreases 1 share at time t=10
>
> So,
>
> - Between time t=0 and t=1, only user A is getting the rewards, so A
>   gets 2.778 USDT
> - Between time t=1 and t=10, A has 2 shares, while B has 1 share, so A
>   gets 2/3 of what's distributed and B gets 1/3: A gets 16.667 USDT and
>   B gets 8.333 USDT
> - Between time t=10 and t=3600, A and B have 1 share each, so each gets
>   1/2 of what's distributed: 4986.111 USDT each
>
> So, in total, A gets 5005.556 USDT and B gets 4994.444 USDT.
>
> Expected output: `{"A": 5005.556, "B": 4994.444}`

The signature in the task shows `tuple[str,int]`; the events it
describes have three fields, which is what `index.py` accepts. The
statement mentions both "10,000 USDT" and "1000 tokens"; the code uses
the rate, 2.778 USDT/sec for 3600 seconds.

## License

GNU General Public License v3.0. See [LICENSE](LICENSE).
