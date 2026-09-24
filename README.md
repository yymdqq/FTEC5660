# FTEC5660 Homework 1: Receipt Chain

Build a LangChain pipeline that reads every supermarket receipt in a folder
with the vision-capable DeepSeek Flash model and answers these two questions:

1. How much money did I spend in total for these bills?
2. How much would I have had to pay without the discount?

For this homework, **amount spent** means the final payment after the receipt's
rounding line. **Without the discount** means the sum of the original positive
item prices: add back every promotion, coupon, member, app, packaging-damage,
and percentage discount, but do not add back rounding.

## Student task

Only edit the two functions in `hw1.py` that contain `### YOUR CODE HERE`:

- `build_chain()` creates your LangChain chain.
- `answer_queries()` runs the chain on the receipt images and returns one final
  response for each question.

You may use prompt chaining, routing, parallel calls, reflection, or a
combination. Your final responses should each contain one HKD amount. Do not
hard-code filenames or public answers; grading uses unseen receipt folders.

## Setup and public test

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Put your DeepSeek key after `DEEPSEEK_API_KEY=` in `.env`, then run:

```bash
python3 hw1.py --image-folder public_test
```

The program creates `results.csv` in the current directory. Its columns are
`query`, `model_response`, and `correctness`. The public answers are in
`public_test/ground_truth.json`. The starter intentionally returns the dummy
response `please design your chain to answer these two queries.` so it runs
before you add any API code.

The required model is `deepseek-v4-flash-vision-exp`, the vision-capable
DeepSeek Flash model. JPEG, PNG, GIF, and WebP inputs are accepted by the
homework runner.


## Homework 1 solution: 
> to students: please fill your solution description here.
1. Solution Approach

The goal of this homework is to read all supermarket receipts in a folder and answer two questions:

How much money did I spend in total?
How much would I have paid without the discounts?

My solution uses LangChain with the DeepSeek Vision model. First, I create a prompt using ChatPromptTemplate and use the required deepseek-v4-flash-vision-exp model. For each receipt image in the folder, the local image is converted into a Base64 data URL and then passed to the vision model.

Instead of asking the model to directly calculate the final totals for the whole folder, I ask the model to extract two amounts from each individual receipt:

paid: the final amount actually paid after rounding.
without_discount: the receipt subtotal plus all discounts, promotions, and coupons added back as positive amounts, without adding back the rounding amount.

The model returns these two values in a simple JSON format for each receipt. Python then uses Decimal to sum the values from all receipts. Finally, the two totals are formatted as HK$xx.xx and returned to the provided grading code.

2. Chain Workflow

<img width="2231" height="485" alt="11" src="https://github.com/user-attachments/assets/2cf858d7-0693-4ef2-a650-81b5c36e594d" />

3. Per-Receipt Processing

For each receipt, the model identifies the final payment amount and the amount that would have been paid without discounts.

For example, if a receipt contains:

Items:       $10.00 + $36.90 + $60.80
Discount:    -$5.39
SUBTOTAL:    $102.31
ROUNDING:    -$0.01
OCTOPUS:     $102.30

the expected extraction is:

{
    "paid": 102.30,
    "without_discount": 107.70
}

The calculation is:

paid = 102.30

without_discount
= 102.31 + 5.39
= 107.70

The rounding amount is not added back to without_discount.

4. Final Aggregation

After processing all receipt images, Python uses Decimal to calculate the final totals. This design separates receipt understanding from numerical aggregation. The vision model is responsible for reading and extracting information from the receipt images, while Python is responsible for summing the extracted monetary values. This reduces the risk of arithmetic errors when multiple receipts are processed and also makes the intermediate results easier to inspect.



