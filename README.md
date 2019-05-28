# LendingClub Profitability

Visualizing loans and building investment models


### What Is LendingClub?

[LendingClub](https://www.lendingclub.com/) is a peer-to-peer lending company. All loans have either a 36- or 60-month term, with fixed interest rates and equal payments. Each loan is broken up into many Notes, which corresponds to a fraction of the overall loan in amounts as low as $25. Investors build their portfolio by purchasing/investing in many Notes from different loans and borrowers.

Loans offered by LendingClub are a form unsecured debt like credit card debt.

### Dataset

The [LendingClub 2016-2018 dataset](https://www.lendingclub.com/info/download-data.action) contains on average **114k** loans per quarter.

Below are visualizations of the dataset:

![LendingClub 2016-2018 - Total Current Loan Amount Per State](assets/LendingClub&#32;2016-2018&#32;-&#32;Current&#32;Loan&#32;Amount&#32;Per&#32;State.gif)

No surprise that [more populous states](assets/US&#32;Population&#32;2016-2018.gif) borrow more.

![LendingClub 2016-2018 - Total Loan Amount By Loan Status](res/LendingClub&#32;2016-2018&#32;-&#32;Total&#32;Loan&#32;Amount&#32;By&#32;Loan&#32;Status.png)

Since all loans have either a 36- or 60-month term, **Current** loans rollover quarter-to-quarter. Unfortunately, LendingClub has omitted the **id** attribute for us to track loans over time and calculate the rollover.

![LendingClub 2016-2018 - Current Loan Amount By Grade](res/LendingClub&#32;2016-2018&#32;-&#32;Current&#32;Loan&#32;Amount&#32;By&#32;Verification&#32;Status&#32;Stacked&#32;Area&#32;Plot.png)

Loan applicants may provide income verification to help investors better understand risk. Verification statuses are defined below:  
- **Verified**: loan applicant submitted documents such as paystubs, W-2 forms, or other tax records to verify their income.
- **Source Verified**: LendingClub electronically checked the loan applicant's income data through a third-party.

More information about income verification can be found [here](https://www.lendingclub.com/investing/investor-education/income-verification).

![LendingClub 2016-2018 - Current Loan Amount By Grade](res/LendingClub&#32;2016-2018&#32;-&#32;Current&#32;Loan&#32;Amount&#32;By&#32;Grade&#32;Stacked&#32;Area&#32;Plot.png)

Loan grades range between **A** to **G**. Each grade is further categorized into five subgrades.   
More information about how LendingClub calculates loan grade can be found [here](https://www.lendingclub.com/foliofn/rateDetail.action).

![LendingClub 2016-2018 - Current Loan Amount By Grade](res/LendingClub&#32;2016-2018&#32;-&#32;Current&#32;Loan&#32;Amount&#32;By&#32;Title&#32;Stacked&#32;Area&#32;Plot.png)

Loans are categorized into **14** different titles/purposes. 

### Methodology

The goal is to distinguish between **Fully Paid** or **Default**/**Charged Off** loans. Fully Paid will correspond to the positive class. Default/Charged Off will correspond to the negative class. Other loan statuses are omitted because we can't track the rollover. The same loan could be present in several quarters, which could skew the analysis.

#### Preprocessing

Below are the preprocessing steps:
1. The quarterly loans from 2016 to 2018 are combined. We keep only the following **10** attributes: "loan_amnt", "int_rate", "term", "grade", "sub_grade", "installment", "annual_inc", "loan_status", "verification_status", and "purpose".
2. Outliers were removed where annual income equaled 0 or greater than 1 million.
3. Categorical variables were encoded into binary vectors e.g. verification status and purpose.
4. Ordinal variables were encoded their numerical values e.g. grade and subgrade.

In total, ~380k loans were Fully Paid, while ~108k loans were Default/Charged Off. 

To maintain this ~3.5:1 loan ratio, **28k** Fully Paid loans and **8k** Default/Charged Off loans were randomly sampled and set aside as the **validation set**.

**200k** loans were randomly sampled, evenly split between the positive and negative class. This will be our **training/test set**.

#### Models

Four classifiers were selected, the latter two being ensemble classifiers:
- SVC / Linear SVM
- Decision Tree
- Random Forest
- Gradient Boost

An 80/20 split was created for the training/test set.

Since SVMs are sensitive to scaling, the data was scaled to [0, 1] using a MinMaxScaler.

For the other classifiers, the data was scaled using StandardScaler though this step could be skipped. 

Each model had its hyperparameters tuned using GridSearchCV/RandomSearchCV. Cross validation was set to 5. The classifiers were evaluated using f1-score.

The best classifier was retrained on the whole training set before scoring the test set on accuracy. 

### Results

Classification results are shown below:

|                      | svm | decision_tree | random_forest | gradient_boosting |
|----------------------|-----|---------------|---------------|-------------------|
| train_score_f1       | 123 | 123           | 123           | 123               |
| test_score_acc       | 123 | 123           | 123           | 123               |
| validation_score_acc | 123 | 123           | 123           | 123               |

The trained models were then used to prediction the class probability of the validation set. This probability represented the model's confidence that a given loan was Fully Paid or Default/Charged Off. A given loan would only be invested if the predicted confidence was greater than or equaled a confidence threshold. 

A confidence threshold = 0 means that we will invest in all available loans.   
A confidence threshold = 0.5 means that we will invest in all available loans where confidence was greater than or equal to 50.

![LendingClub 2016-2018 - Total Percentage of Loans Invested On Validation Set](res/Prediction/LendingClub&#32;2016-2018&#32;-&#32;Total&#32;Percentage&#32;of&#32;Loans&#32;Invested&#32;On&#32;Validation&#32;Set.png)

As our confidence threshold approaches 1.0, the number of available loans for us to invest decreases.

![LendingClub 2016-2018 - 5 Year Total ROI On Validation Set](res/Prediction/LendingClub&#32;2016-2018&#32;-&#32;5&#32;Year&#32;Total&#32;ROI&#32;On&#32;Validation&#32;Set.png)

Since all loans have either a 36- or 60-month term, we can calculate the 5-year total return on investment (ROI) as **(total_return - capital_invested) / capital_invested**, where:
- capital_invested = sum(loan_amnt)
- For Fully Paid loans, total_return += installment * term.
- For Default/Charged Off loans, total_return += -1 * loan amount.

As we approach confidence threshold = 1.0, the ROI will be more sporadic and volatile because there's less available loans to invest. Eventually, no loans are invested because no loans satisfy the confidence threshold. 

Keep in mind that calculation for the 5-year total ROI fully invests in each loan. This is important because the loan amount will vary for each loan.

![LendingClub 2016-2018 - Current Loan Amount By Grade](res/Prediction/LendingClub&#32;2016-2018&#32;-&#32;Amortized&#32;Per&#32;Dollar&#32;Annual&#32;ROI&#32;On&#32;Validation&#32;Set.png)

We can amortize the ROI using a per dollar metric. Returns are normalized by term and loan amount. This means that the dollar amount is invested equally across the available loans.

![LendingClub 2016-2018 - Current Loan Amount By Grade](res/Prediction/LendingClub&#32;2016-2018&#32;-&#32;Percent&#32;Positive&#32;In&#32;Confidence&#32;Threshold&#32;On&#32;Validation&#32;Set.png)

We can also check for each confidence threshold what percentage of loans were actually Fully Paid. The horizontal red line represents the ~3.5:1 ratio of Fully Paid:Default/Charged Off loans in the validation set and overall dataset. Values above this horizontal red line represents the degree to which the model selected loans that are Fully Paid above baseline loan ratio. The more above the line the better.

### Conclusion

While the potential fesibility of LendingClub for an investor is left as an exercise for the reader :), further back-testing should be employed in the spirit of robustness. The results shown in the previous section do not account for external factors that could influence returns such as interest rate, employment rate, geopolitical events, and etc. These factors are important because loans offered by LendingClub are a form unsecured debt like credit card debt. When the economy weakens, unsecured debt is the first type of debt borrower stop repaying.

In addition, the diversity and popularity of low-cost ETFs provide investors with many investment options to consider. These ETFs also provide investors liquidity, which LendingClub does not. If an investor chooses to invest in LendingClub Notes, the money is tied until the end of the loan. Potential investors will need to carefully consider this issue of liquidity.

The ROI shown in the Results section does not account for the dollar amount available to invest. Further strategies should account for this available capital and test for the diversification of Notes vs. selecting quality Notes.

The models can be further improved if we had a better understanding of the loan applicant's financial situation. The original dataset does include columns that help us understand just that, but many row contain null values. 

### Disclaimer

The information contained herein (together, "Content") is presented for educational and informational purposes only, on an "as is" basis. No warranty of any kind, expressed or implied, that the Content is accurate, complete or error-free, and it should not be relied upon as such. Nothing in the Content shall constitute or be construed as an offering of financial instruments, or as investment advice, credit analysis or recommendations of an investment strategy or whether or not to "buy", "sell" or "hold" an investment. Ideas and strategies should never be used without first assessing your own personal and financial situation, or without consulting your financial professional.