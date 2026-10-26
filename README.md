# SQL Analysis: Wholesale Model Sales and IMDB Movie Ratings

How do you turn plain business questions into reliable SQL? This project answers eight stakeholder requests against a wholesale scale-model company database, then explores an IMDB movie database to ask whether longer films earn higher ratings. Every query is run through `sqlite3` and returned as a Pandas DataFrame, and the key results are visualised with Matplotlib.

**Short answer:** two customers account for about half of the revenue among the top 10 payers, product variety tracks sales volume almost perfectly across product lines, and movie ratings dip for films of 60 to 89 minutes before climbing steadily as runtime grows. The value of this project is in the query design, the choice of chart for each question, and the honest reading of what each result does and does not show.

## Problem Statement

**Part 1 (guided).** A wholesale seller of miniature cars, motorcycles, planes and ships wants reports on its customers, products and offices so it can target promotions, recognise its best customers and plan its product range and workforce.

**Part 2 (open exploration).** A movie analyst wants to know what the IMDB data for 2010 to 2019 can say about the link between runtime and audience rating, and which cleaning steps are needed before a deeper analysis.

## Datasets

| Part | Source | Contents |
|---|---|---|
| 1 | `data.sqlite` (wholesale model company database) | Customers, orders, order details, payments, products, product lines, employees and offices |
| 2 | `im.db` (IMDB extract, unzipped from `im.db.zip`) | 8 tables: `movie_basics`, `movie_ratings`, `movie_akas`, `directors`, `writers`, `principals`, `persons`, `known_for` |

- **Part 2 scope:** movies released between 2010 and 2019. Records with a `start_year` after 2019 are excluded because that data is incomplete.
- **Part 2 size:** 145,081 movies fall in the 2010 to 2019 window.
- **Missing data:** 30,765 of those movies (about 21%) have no runtime and 5,359 have no genre. Of the 73,856 rated movies, about 61% have fewer than 100 votes. Null values were excluded from the exploration, not filled in.

## Approach

1. **Connect** to each database with `sqlite3` and read every query result into a named Pandas DataFrame.
2. **Filter and pattern match** with `WHERE`, `LIKE` and wildcards (Steps 2 and 3).
3. **Aggregate** with `GROUP BY`, `AVG`, `SUM` and `COUNT` (Steps 4 and 5).
4. **Join** customers, orders, order details, products and product lines, using `HAVING` to filter on aggregates (Steps 5 to 7).
5. **Decompose** a problem with a subquery to find small offices before pulling their employees (Step 8).
6. **Visualise** three of the results, picking a chart type to match each question: a bar chart for state averages, a horizontal bar chart for a ranking, and a scatter plot for a relationship.
7. **Explore** the IMDB database: inspect the schema, count records, compare genres, then bucket runtimes with a `CASE` expression and test them against average rating.
8. **Reflect** in writing on the `WHERE` clause, the chart choice and the subquery, explained for a non-technical reader.

## Key Results

### Part 1: Customer and product queries

| Step | Question | Technique | Result |
|---|---|---|---|
| 2 | California customers with a credit limit above 25,000 | `WHERE` with `AND`, `CAST` | 10 customers, led by Mini Gifts Distributors Ltd. (210,500), then Collectable Mini Designs Co. and Corporate Gift Ideas Co. (105,000 each) |
| 3 | International customers with "Collect" in the name | `LIKE '%Collect%'` | 15 customers across 11 countries, from Australia to Switzerland |
| 4 | Average credit limit by US state | `GROUP BY`, `AVG` | Highest in NH (114,200), lowest in NJ (43,000) |
| 5 | Top 10 customers by total payments | `JOIN`, `SUM`, `LIMIT` | Euro+ Shopping Channel (715,739) and Mini Gifts Distributors Ltd. (584,188) lead |
| 6 | Customer and product pairs with 10 or more units | `JOIN` x3, `HAVING` | 2,531 customer-product pairs, up to 308 units |
| 7 | Quantity ordered and product count per line | `COUNT(DISTINCT)`, `SUM` | Classic Cars lead with 37 products and 35,582 units |
| 8 | Employees in offices with fewer than 5 staff | Subquery with `HAVING` | 12 employees across 5 offices, with job titles and supervisors |

![Average credit limit by US state](images/01_avg_credit_limit_by_state.png)

- **Credit limits vary widely by state.** New Hampshire's average is more than 2.6 times New Jersey's. But NH, NJ and NV each have only one customer, so those averages are a single credit limit, and a state average is a starting point for policy, not a rule.
- **`creditLimit` is stored as text** in `data.sqlite`. A plain `creditLimit > 25000` comparison treats every value as larger than the number and sorts alphabetically, which wrongly lets an 11,000 limit through. Casting the column with `CAST(creditLimit AS REAL)` fixes both the filter and the sort.
- **Revenue is concentrated.** The top two customers paid about 1.30 million in total, roughly 52% of the 2.52 million paid by the top 10 combined.

![Top 10 customers by total payments](images/02_top_10_customers.png)

- **Variety and volume move together.** Across the seven product lines, the number of products and total units ordered have a correlation of about 0.9998. Each line sells roughly 940 to 1,000 units per product, so a line's volume is almost fully explained by how many products it carries.

| Product line | Products | Units ordered | Units per product |
|---|---|---|---|
| Classic Cars | 37 | 35,582 | 962 |
| Vintage Cars | 24 | 22,933 | 956 |
| Motorcycles | 13 | 12,778 | 983 |
| Planes | 12 | 11,872 | 989 |
| Trucks and Buses | 11 | 11,001 | 1,000 |
| Ships | 9 | 8,532 | 948 |
| Trains | 3 | 2,818 | 939 |

![Number of products vs total quantity ordered](images/03_products_vs_quantity.png)

### Part 2: Do longer movies get higher ratings?

Movies from 2010 to 2019 with a recorded runtime and rating, restricted to titles with at least 100 votes:

| Runtime | Movies | Average rating |
|---|---|---|
| Under 60 min | 280 | 6.62 |
| 60 to 89 min | 7,607 | 5.54 |
| 90 to 119 min | 15,708 | 5.93 |
| 120 to 149 min | 3,455 | 6.46 |
| 150+ min | 928 | 6.58 |

![Average IMDB rating by runtime bucket](images/04_runtime_vs_rating.png)

- **The relationship is not a straight line.** Ratings are lowest for 60 to 89 minute films, then rise by about one full point by 150 minutes or more.
- **The under 60 minute bucket is small** (280 films) and probably a different kind of content, such as shorts and documentaries, so it should not be read as "short films are best."
- **The genre scan** surfaced documentary-heavy combinations at the top, but many of those rest on very few votes (some average under 60 votes per film), so they are weak evidence.

**Business question for further analysis:** do films of 120 minutes or more earn consistently higher IMDB ratings than shorter films, and at what runtime does the effect appear, once genre and vote count are controlled for?

**Data cleaning tasks identified:**

1. About 21% of movies have a null `runtime_minutes` and must be excluded before any runtime analysis.
2. About 61% of rated movies have fewer than 100 votes, which distorts average ratings, so a minimum vote threshold is needed.
3. Genres are stored as comma-separated combinations (for example `Action,Crime,Drama`) and should be split into one genre per row.
4. The 1,063 records with `start_year` after 2019 must be filtered out consistently in every query.

## Conclusions

- Each SQL tool fits a different kind of question: `WHERE` and `LIKE` to select, `GROUP BY` to summarise, `JOIN` to combine tables, `HAVING` to filter on aggregates and subqueries to split a problem into two steps.
- The chart should follow the question. A bar chart compares categories, a horizontal bar chart suits long customer names in a ranking, and a scatter plot is the right way to ask whether two numbers are related.
- In this company, product range drives volume. Lines with more products sell proportionally more, which supports expanding the lines that already sell well.
- In the movie data, runtime and rating are related, but not in a simple "longer is better" way. A mid-length dip and a rise for long films need genre and vote controls before anyone acts on them.

## Limitations and Next Steps

- **State averages rest on few customers.** Three of the eight states have a single customer. Adding a customer count to Step 4 would show which averages can be trusted.
- **Column types were not enforced.** Because `creditLimit` is text, numeric filters and sorts need a `CAST`. Converting the column once at load time would be cleaner.
- **Revenue was measured by payments, not orders.** Joining order totals would separate money received from money owed.
- **Seven product lines is a small sample** for the products-versus-volume comparison, so the near-perfect correlation should be treated as descriptive.
- **The runtime analysis is descriptive.** It compares bucket averages and does not test whether the differences are statistically significant.
- **Genre was not split** into individual genres, so combinations such as `Action,Adventure` are treated as one category.
- **A vote threshold was applied only to the runtime query,** not to the genre scan.
- **Next steps:** split genres into rows, add significance tests and confidence intervals, and extend the question to directors, budgets or release year.

## Tech Stack

Python, SQL (SQLite), pandas, Matplotlib, seaborn, Jupyter

## How to Run

```bash
git clone https://github.com/ElvisOmondi7/dsc-sql-summative-lab.git
cd dsc-sql-summative-lab.git
pip install pandas matplotlib seaborn jupyter
jupyter notebook SQLSummativeLab.ipynb
```

`data.sqlite` and `im.db.zip` need to sit in the same folder as the notebook. The notebook unzips `im.db.zip` automatically for Part 2.

## Repository Contents

```
.
├── SQLSummativeLab.ipynb   # Full analysis notebook (Part 1 and Part 2)
├── data.sqlite             # Wholesale model company database (Part 1)
├── im.db.zip               # IMDB movie database (Part 2)
├── images/                 # Charts used in this README
└── README.md
```

## Author

**Elvis Omondi Okoth**, Data Science and Statistics, Nairobi, Kenya

[LinkedIn](https://www.linkedin.com/in/elvis-omondi-200a06264) | okothelvisomondi@gmail.com | [GitHub](https://github.com/ElvisOmondi7)

Completed as a summative lab in the Data Science program at Moringa School.
