# LendingClub Profitability

Visualizing loans and building loan investment models


### What Is LendingClub?

LendingClub is a peer-to-peer lending company. All loans have either a 36- or 60-month term, with fixed interest rates and equal payments. Each loan is broken up into many Notes, which corresponds to a fraction of the overall loan in amounts as low as $25. Investors build their portfolio by purchasing/investing in many Notes from different loans and borrowers.

Loans offered by LendingClub are a form unsecured debt like credit card debt.

### Dataset

The [LendingClub 2016-2018 dataset](https://www.lendingclub.com/info/download-data.action) contains on average **114k** loans per quarter.

Below are visualizations of the dataset:

![LendingClub 2016-2018 - Total Current Loan Amount Per State](assets/LendingClub&#32;2016-2018&#32;-&#32;Current&#32;Loan&#32;Amount&#32;Per&#32;State.gif)

No surprise that [more populous states](assets/US&#32;Population&#32;2016-2018.gif) borrow more.

![LendingClub 2016-2018 - Total Loan Amount By Loan Status](res/LendingClub&#32;2016-2018&#32;-&#32;Total&#32;Loan&#32;Amount&#32;By&#32;Loan&#32;Status.png)

Since all loans have either a 36- or 60-month term, **Current** loans rollover quarter-to-quarter. Unfortunately, LendingClub has omitted the **id** attribute for us to track loans over time and calculate the rollover.

![LendingClub 2016-2018 - Current Loan Amount By Grade](res/LendingClub&#32;2016-2018&#32;-&#32;Current&#32;Loan&#32;Amount&#32;By&#32;Verification&#32;Status&#32;Stacked&#32;Area&#32;Plot.png)

Loan applications may provide income verification to help investors better understand risk. Verification statuses are defined below:  
- **Verified**: loan application submitted documents such as paystubs, W-2 forms, or other tax records to verify their income.
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
3. Non-ordinal categorical variables were encoded into binary vectors e.g. verification status and purpose.
4. Ordinal variables were encoded their numerical values e.g. grade and subgrade.

In total, ~380k loans were Fully Paid, while ~108k loans were Default/Charged Off. 

To maintain this ~3.5:1 loan ratio, **28k** Fully Paid loans and **8k** Default/Charged Off loans were randomly sampled and set aside as the **validation set**.

**200k** loans were randomly sampled, evenly split between the postive and negative class. This will be our **training/test set**.

#### Models

Four classifers were selected, the latter two being ensemble classifiers:
- SVC / Linear SVM
- Decision Tree
- Random Forest
- Gradient Boost

An 80/20 split was created for the training/test set.

Since SVMs are sensitive to scaling, the data was scaled to [0, 1] using a MinMaxScaler.

For the other classifiers, the data was scaled using StandardScaler though this step could be skipped. 

Each model had its hyperparameters tuned using GridSearchCV/RandomSearchCV. Cross validation was set to 5. The classifers were evaluated using f1-score.

The best classifer was retrained on the whole training set before scoring the test set on accuracy. 

### Results



### Disclaimer

The information contained herein (together, "Content") is presented for educational and informational purposes only, on an "as is" basis. No warranty of any kind, expressed or implied, that the Content is accurate, complete or error-free, and it should not be relied upon as such. Nothing in the Content shall constitute or be construed as an offering of financial instruments, or as investment advice, credit analysis or recommendations of an investment strategy or whether or not to "buy", "sell" or "hold" an investment. Ideas and strategies should never be used without first assessing your own personal and financial situation, or without consulting your financial professional.