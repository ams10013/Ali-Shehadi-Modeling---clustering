# Gym member segments: who should get the monthly offer?

k-means clustering of gym members into segments, used to decide which segments get the $10 extras credit.

## Files

| File | What it is |
|---|---|
| `analysis.ipynb` | All code and output. Run top to bottom to reproduce every number and chart. |
| `data/gym_members.csv` | 330 members, averages over the last 3 months |
| `figures/elbow_train_test.png` | Within-group spread vs k, training members and test members |
| `figures/monthly_net.png` | Gym's monthly net from the credit, baseline vs v1 |

## How to run

```bash
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace analysis.ipynb
```

The split and k-means both use `random_state=42`, so results are identical on every run.

## Method

1. Random split: 231 training members (70%), 99 test members (30%).
2. Features: age, visits per month, minutes per visit, classes per month, months as a member, monthly extras spend. `member_id` is left out. Features are standardized with a scaler fit on the training members only.
3. k-means for k = 1 to 8 on the training members. The elbow is at k = 4.
4. Test check without refitting: each test member is assigned to the nearest training center and the squared distances are summed for each k.
5. Credit decision from finance's assumptions, applied member by member: a member spending under $20 a month on extras nets the gym +$5 from the credit, a member spending $20 or more costs $10. A segment gets the credit if its members net positive.

## Results

**Spread (sum of squared distances)**

| | k = 1 (baseline) | k = 4 (v1) | Cut |
|---|---|---|---|
| Training members | 1,386 | 336 | 76% |
| Test members | 601 | 150 | 75% |

Both curves drop sharply up to k = 4 and flatten after, so the segments hold up on members the model never saw.

**Segments (centers, all 330 members assigned)**

| | A | B | C | D |
|---|---|---|---|---|
| Members | 70 | 109 | 91 | 60 |
| Age | 27 | 33 | 43 | 64 |
| Months as member | 6 | 30 | 24 | 58 |
| Visits / month | 3 | 16 | 9 | 12 |
| Minutes / visit | 41 | 65 | 55 | 47 |
| Classes / month | 1 | 1 | 7 | 4 |
| Extras spend / month | $9 | $25 | $47 | $17 |
| Share spending $20+ | 16% | 61% | 99% | 38% |
| Net if given credit | +$185 | −$445 | −$895 | −$45 |
| Gets credit | Yes | No | No | No |

New member (age 30, 4 visits, 42 minutes, 1 class, 5 months, $10 extras): segment A, gets the credit.

**Monthly net from the credit**

| | Members getting credit | Monthly net |
|---|---|---|
| Baseline: credit to everyone | 330 | −$1,200 |
| v1: credit to segment A only | 70 | +$185 |

Targeting improves the monthly net by $1,385, about $16,600 a year.
