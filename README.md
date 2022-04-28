# The Data
Overall, we are trying to compare the numbers at D0 to the numbers at D1 and see what differences in those numbers are not accounted for by the transactions that took place over the course of the day. To do this, I've converted the incoming space-separated recon.in data into different pandas dataframes. After processing the transaction data to turn it into a summary of changes to either the stocks or the total amount of cash, we can merge the data into one dataframe and quickly check for the discrepancies that we can export and report in our recon.out file.


# Main Steps
1. Process the recon.in data to create 3 dataframes for D0-POS, D1-TRN, and D1-POS
2. Convert the D1-TRN data into a dataframe containing the stock name and the net change to the shares or the cash value
3. Merge the D0-POS, D1-POS, and the converted D1-TRN dataframes into one dataframe.
4. Calculate the recon.out using data in the dataframe; select the stock names and reconciliation values (that are not 0) and convert them into space-separated data in a txt file.

## Step 1
- Install pandas as pd
- Write a function `get_line_number` that takes the parameters `string` and `data` and from them finds the line number of that string.
- Use the `get_line_number` function to find the value of `skiprows` and the cutoff of `nrows` for each invocation of `pd.read_csv` so that we can process the 3 different dataframes (this is necessary because the D1-TRN data has a different shape than the D0-POS and D1-POS data so we cannot process it all at once)
- Use `pd.read_csv` with the aforementioned `skiprows` and `nrows` with `delimiter=' '` to convert the space-separated data into different pandas dataframes.
- Assign standardized column names ('stock', 'shares_0' / 'stock', 'action', 'shares', 'value' / 'stock', 'shares_1') to make the data easier to manipulate and merge.

## Step 2
- Set up a dictionary called `cash_actions` for which the keys are the strings of the different actions (ex: 'BUY') and the values are -1 or 1 depending on whether that action increases or decreases the amount of cash.
- Declare an empty dictionary called `change_obj` and initialize a variable called `cash_change` to 0.
- Use a for loop to iterate through the D1-TRN dataframe values
    - If the stock name is not already in `change_obj`, add it as a key with the value of `-1 * cash_actions[*action*] * *shares*`
    - Otherwise, if it is in `change_obj`, add that value to the current value stored in the dictionary
    - Increment `cash_change` by `cash_actions[*action*] * *value*`
- Add a `'Cash'` key to `cash_obj` with the value of `cash_change`
- Create a data from from `cash_obj` with the columns `'stock'` and `'change'`

## Step 3
- Using an outer join with `pd.merge`, merge the dataframes for D0-POS, D1-POS, and the new dataframe from the D1-TRN on the `'stock'` column.
- Use the `.fillna(0)` to fill in any missing values in the merged dataframe.

## Step 4
- Add a new column to the dataframe called `recon_out`. Calculate it by subtracting the values in `shares_0` and `change` from `shares_1` to find any discrepancies in the transactions.
- Create a new dataframe that contains the stock name and the `recon_out` value where the `recon_out` value is not zero (i.e. there is a discrepancy).
- Use `.to_csv` with a `sep=' '` to convert the dataframe to a space-separated txt file called recon_out.txt. Set the `index` and the `header` to false.
