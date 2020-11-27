---
layout: 10_topic
title: Data Science
permalink: /comparetwolists
---


# Compare Two Lists - Excel vs. Python
<br/><br/>
Comparing two lists is a problem that frequently occurs.
<br/><br/>
In my case, I got two lists in that I can examine in Excel.
<br/><br/>
 Certainly, this problem can be solved remaining in Excel. If you did so yourself, you might know to pain to get dizzy at looking at all these IDs, worrying of having made a mistake when expanding those columns - and to start all over again, when doubts occur on the correctness of your results or when the input data change a little bit.
<br/><br/>
Using Python, you can solve this problem fast, robust and with a essentially **3 lines of coding**.
<br/><br/>
The Python coding outlined below run within 1 second for 100.000 IDs - returning all the info you want, ready to be used as Excel.
<br/><br/>
The difference is made by just a few Python and Python Pandas commands.
<br/><br/>
I will show you here how I solved the problem. First in Excel, then in Python. 
<br/><br/>
Your respective solution in Excel might be much better than mine, but still you might want to compare it with the subsequent Python solution. I will also show how to get from Excel to Python and back.
<br/><br/>
After having gone through both solutions, we will summarize and discuss their respective benefits.

## Use Case

Let's assume you are responsible for the checking of invoices.
You hold a list of accounts from which you expect invoices. We name this list: A. It holds all of 'your' accounts.

You get in a list of invoices, each coming from an account. Each account has a unique account ID. We name this list: B.
<br/><br/>
The questions you ask are as follows:

- Did I recieve invoices for all of my accounts?
- Did I recieve invoices for accounts that are not mine?

For me, this use case came in recently and at first surprisingly after the latest re-organization. Suddenly, expected invoices were missing - and doubt came up, if invoices from other accounts were added. Next thing I wrote was a predecessor to the example here that checks, if I got exactly those accounts on my invoice list that I had expected.
Possibly, the list with my accounts has become outdated, but I still need those differences to discuss with colleagues, to open tickets and to adapt my own lists after respective decisions.
Soon after, I realized that it is not only me having this problem, but all my peers dealing with such a task of invoice checking are facing the same problem. Suddenly, all 'internal lists' are re-shuffled. This problem also reoccurs with each reorganization that implies a re-distribution of such accounts.

Wouldn't it be time to work out the pattern and to come up with an efficient solution?

## Specification of Problem and Solution

For the example here, I created IDs representing accounts of the following pattern:

<br/>
>
        "5cgb19c9-nxs2-c9f-xx12-894w1b894f38"

<br/>
They are in fact of identical format as the 'real accounts' that I am dealing with.

I tried various amounts of IDs and ended up with 30 IDs, which seem to be sufficient to illustrate the points to be made here. No need to deal with my typical amounts of 80 to 800 account IDs.

Using simple random numbers, I distributed those 30 IDs either to list A, to list B or to both lists.

As a solution, I would like to have 3 lists, let's call them, S1, S2, S3.

- S1: IDs that occur in both A and B
- S2: IDs only occurring in A
- S3: IDs only occurring in B

The problem shall be called solved, when the three lists S1, S2 and S3 are known and sufficient sample tests using search and find assure us of their correctness. Example: Take any ID from list S2 and check that it only occurs in list A, but not in list B.

## A Solution in Excel

As a starting point, let's assume we have the two lists A and B in an Excel file each.

I copied the sheet of file A and copied the content of file B into that new sheet. This is the only sheet that I am working with here.
In the picture below, you see the two lists as the two left-most columns, which I labelled for more clarity as 'List_A' and 'List_B'.

The common technical approach that I also have seen my colleagues doing is to use the "vlookup" command of Excel. My solutions requires as a next step an intermediate column for this vlookup command. In this intermediate column, I get all the IDs that are in list B, but not in list A. The formula looks like this:

<br/>
>

    =VLOOKUP(C3;B$3:B$30;1;FALSE)

<br/>
There are several traps, you might run into when using this command.

- First of all, in our case here you need to set the last parameter to FALSE. FALSE here means 'exact match'.

- Next trap is to forget the "$" to fix the range specification of the searched column. If you forget that before expanding the equation to the complete column, the IDs searched get increasingly fewer - leading to a wrong result.
- Finally, you need to have the searched list sorted before you execute the vlookup command.

![Screenshot of Excel solution](./md/10_DataScience/Intersection_Excel.PNG)

Each unique ID in the second column leads to a "#N/A" in the third column. Optically, this is not a beauty, but it helps to select the desired subset by just filter off those '#N/A'. You can then copy and paste the content into the next column and name it rightfully S1, as it contains the shared IDs.

Repeating the filter for the complementary subset, directly leads to S3, i.e. the unique entries in list B.
Getting to the unique entries in list A, however, the procedure using vlookup has to be repeated, this time with "entries in A, but not in B". Similarly to S3, the list S2 can be obtained, which contains the unique entries in A.

Ok. We are done for the Excel solution, with S1, S2 and S3 being populated and sample checks being all good.

## A Solution in Python

We start here, too, with lists A and B.

The steps used here are as follows:

- Merge the two list A and B with an inner join: Get S1
- Concat list A with S1 and delete duplicates: Get S2
- Concat list B with S1 and delete duplicates: Get S3

Here, we need to introduce Pandas which is an library for Python. Pandas is usually imported - after installation - to the respective Python program and abbreviated as "pd". Pandas comes with so-called dataframes which hold lists.
In my example, A and B are already such dataframes, but they can also be the result of an import from Excel files - after having installed the library xlrd - using, for instance,

<br/>
>
    A = pd.read_excel(fileAbsPath, index_col=0)

<br/>

where fileAbsPath is the absolute path to the Excel file. Details are given in the sources linked in the references.

In Python, the command used for merging the two dataframes is as follows:

<br/>
>
    S1 = pd.merge(A, B, how="inner", left_on="AccountID", right_on="AccountID")

<br/>
In this merge, the "inner" value for the "how" parameters specifies to only merge those rows which have the same value for AccountID in both the "left" and the "right" table. It is crucial to ensure that this "inner join' is carried out.

Lists A and B, respectively are added to the new list S1 using concatenate. Afterwards, all IDs that are not unique are deleted.

<br/>
>
    S2 = pd.concat([S1, A], axis='index').drop_duplicates(keep=False)
>

    S3 = pd.concat([S1, B], axis='index').drop_duplicates(keep=False)

<br/>
All entries that only occur in A can only occur only once in table S1 - as S1 contains only those that also occur in B. Deleting all entries that have more than one occurrence thus leaves S2 with only the subset available in table A only.

Pandas' drop_duplicates command does not need the tables to be sorted. This is a nice feature - compared to a "delete_adjacent_duplicates" command - which also happens to makes this coding much more concise.

You see that after 3 lines of code, the tables S1, S2, S3 hold the solution.

All 5 dataframes can easily saved either as 5 Excel files or as one Excel file with 5 sheets. The decisive step is to use the Excel writer:

<br/>
>

    writer = pd.ExcelWriter(fileAbsPath, engine="xlsxwriter")

<br/>

## Combining the solution with the original lists

We walked through the key steps of two independent solutions to come up with the identical result which consisted of holding two lists with unique IDs and one list with common IDs.

In the original problem, two lists, A and B, were connected to other data. Table A in my case to a semantic list of 'my' accounts, holding email addresses of people I know. The other table was connected to invoices.
In the (artificial) example here, this would mean to not having received invoices for 2 accounts, but to have received invoices for 14 out-of-group accounts. How to send back an Excel discussing those 14 accounts?
<br/><br/>
Honestly, I am short of a great answer for Excel. You can of course repeat the vlookup solution again, with the account IDs in the invoices being probed against table S3. You then get again hits and throw out the "#N/A". With hundreds of rows for the invoices this might not a be your solution of choice.

In Python, however, a few lines of additional code solve this problem smoothly. Im my original coding, they are wrapped around the solution above. Before that solution, the original table is copied and truncated to only hold the account IDs. If df_Invoices is the Pandas dataframe that holds all respective invoices, then you get just what was called list B so far using

<br/>
>

    B = df_Invoices["accountID].copy()

<br/>
Finally, the inner join as described above is re-applied.

<br/>
>

    ExcelForDiscussion = pd.merge(S3, df_Invoices, how="inner", left_on="AccountID", right_on="AccountID")

<br/>

Save the dataframe as Excel, attach it to an email. Off it goes. Done.

## Discussion and Contribution

For small data or problems of minor business impact, an Excel solution would be my personal choice. It saves me or you from the work of upload and download to and from Python, when it is an Excel sheet that is needed at the end.
For bigger data - and you get an impression here what 30 IDs can do to your eyes - or important business data, the roundtrip to Python might well be worth the effort. Particularly as the coding in Python is so straightforward.

Each component of the solution used here is well known in the respective community and discussed in much more detail, e.g. in the references given below. However, combining these components to provide the solution to a real-world problem is rarely described in detail.

I want to encourage you to search and detect the pattern discussed here (of having to find an intersection and/or lacking entries).
Hopefully, you get some inspiration from the components used here and my experience in using them.

If you happen to be using better components, a faster process or any other means to improve the solutions outlined here, please be assured of my interest.

## Some References

During this example preparation, I consulted the sites linked below for the solution presented here.

- [Pandas Documentation - Merge](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.merge.html)
- [Pandas Documentation - Concat](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.concat.html)
- [datatofish.com - Read Excel](https://datatofish.com/read_excel)
- [Stackoverflow - python-pandas-find-difference-between-two-data-frames](https://stackoverflow.com/questions/48647534/python-pandas-find-difference-between-two-data-frames=)
- [Stackoverflow - what-is-the-difference-between-using-true-false-1-0-as-the-last-value-of-a-vlookup](https://stackoverflow.com/questions/30219141/what-is-the-difference-between-using-true-false-1-0-as-the-last-value-of-a-vloo)
