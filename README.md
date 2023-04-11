# Federal-Election-project

#### This dataset contains information about the Federal Election 2012, (Candidates, committees, results, and individual and other financial data).

  - `candidates`: candidates registered with the FEC during the 2011-2012 election cycle.
  - `committees`: committees registered with the FEC during the 2011-2012 election cycle.
  - `campaigns`: the house/senate current campaigns.
  - `results_house`: the house results of the 2012 general presidential election.
  - `results_senate`: the senate results of the 2012 general presidential election.
  - `results_president`: the final results of the 2012 general presidential election.
  - `pac`: Political Action Committee (PAC) and party summary financial information.
  - `individuals`: individual contributions to candidates/committees during the 2012 general presidential election.
  - `contributions`: candidates and their contributions from committees during the 2012 general election.
  - `expenditures`: the operating expenditures.
  - `transactions`: transactions between committees.

### Extracting data

```python
fec= pd.read_csv("P00000001-ALL.csv", low_memory=False)
fec.info()
```

![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture.JPG)

## Total donations by party for top occupations?

### Transforming the data

##### It looks like there is missed the party of every candidate in the dataset, so to do further analysis with this information, let us start working on this, then add this information as a column.

```python
unique_cands = fec["cand_nm"].unique()

parties={"Bachman, Michelle": "Republican",
        "Cain, Herman": "Republican",
        "Gingrich, Newt": "Republican",
        "Huntman, Jon": "Republican",
        "McCotter, Thaddeus G": "Republican",
        "Obama, Barack": "Democrat",
        "Paul, Ron": "Republican",
        "Pawlenty, Timothy": "Republican",
        "Perry, Rick": "Republican",
        "Roemer, Charles E. 'Buddy' III": "Republican",
        "Romrey, Mitt": "Republican",
        "Santorum, Rick": "Republican"}

pd.Series(parties, index=['Bachman, Michelle', 'Cain, Herman', 'Gingrich, Newt', 'Huntman, Jon', 'McCotter, Thaddeus G', 'Obama, Barack', 'Paul, Ron', 'Pawlenty, Timothy', 'Perry, Rick', "Roemer, Charles E. 'Buddy' III", 'Romrey, Mitt', 'Santorum, Rick'])

fec["party"]=fec["cand_nm"].map(parties)
fec["party"]
```
![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture2.JPG)

### Cleaning the data

##### Once we have the candidates sorted by their parties we can start applying for some studies like what amount of money each contributor has donated to their party by occupation and employer.

```python
fec= fec[fec["contb_receipt_amt"] > 0]
fec["contbr_occupation"].value_counts()[:10]
```
![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture3.JPG)

##### In order to keep optimizing the data because there are some entries in the contribution employer and contribution information added with different words but with the same meaning that can be transformed to the same word.

```python
emp_mapping = {
    "INFORMATION REQUESTED PER BEST EFFORTS": "NOT PROVIDED",
    "INFORMATION REQUESTED" : "NOT PROVIDED",
    "INFORMATION REQUESTED (BEST EFFORTS)" : "NOT PROVIDED",
    "C.E.O.": "CEO",
    "REQUESTED": "NOT PROVIDED",
    "(RETIRED)": "RETIRED",
    "SELF-EMPLOYED" : "SELF EMPLOYED",
    "'SELF'": "SELF EMPLOYED",
    "(SELF)": "SELF EMPLOYED",
    "-SELF-": "SELF EMPLOYED",
    "SELF": "SELF EMPLOYED",
    "NONE": "NOT PROVIDED"
}

def get_emp(x):
    #if no mapping prrovided, return x
    return emp_mapping.get(x,x)

fec["contbr_employer"] = fec["contbr_employer"].map(get_emp)
fec["contbr_employer"].value_counts()[:10]
```
![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture5.JPG)

### Visualizing the data

```python
by_occupation= fec.pivot_table("contb_receipt_amt", index="contbr_employer", columns="party", aggfunc="sum")
by_occupation.head(50)
```

![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture6.JPG)

```python
over_2mm= by_occupation[by_occupation.sum(axis="columns") > 2000000]
over_2mm
```
![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture7.JPG)

### Total donations by party for top occupations

![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture8.JPG)

## Top donor occupations or top companies that donated to their candidates?

### Transforming the data

##### Another quite interesting study is how much money each candidate has received. We can start analyzing the best seven contributions by contributing occupation and candidate.

##### First we will simplify the database by filtering it by the two most voted candidates Barack Obama and Mitt Romney.

fec_mrbo= fec[fec["cand_nm"].isin(["Obama, Barack", "Romney, Mitt"])]

##### Then we just need to filter by candidate, contribution employer, and the sum of the amount of money received.

```python
def get_top_amounts(group, key, n=5):
    totals=group.groupby(key)["contb_receipt_amt"].sum()
    return totals.nlargest(n)

grouped= fec_mrbo.groupby("cand_nm")
grouped.apply(get_top_amounts,"contbr_employer", n=7)
```
![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture9.JPG)

## Percentage of total donations received by candidates for each donation size?

### Transforming the data

##### Another interesting study is how much each candidate has received by size.
##### To apply this study we need to build an array of size their contributions will be classified in.

```python
bins= np.array([0,1,10,100,1000,10000,100_000,1_000_000,10_000_000])
labels=pd.cut(fec_mrbo["contb_receipt_amt"], bins)
labels

grouped= fec_mrbo.groupby(["cand_nm", labels])
grouped.size().unstack(level=0)

bucket_sums= grouped["contb_receipt_amt"].sum().unstack(level=0)
bucket_sums

normed_sums=bucket_sums.div(bucket_sums.sum(axis="columns"), axis="index")
normed_sums
```
![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture10.JPG)

### Visualizing the data

### Percentage of total donations received by candidates for each donation size.

![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture11.JPG)


## Donation Statistics by State?

##### We can study the amount of donations by state and candidate as well.

```python
grouped= fec_mrbo.groupby(["cand_nm", "contbr_st"])
totals=grouped["contb_receipt_amt"].sum().unstack(level=0).fillna(0)
totals=totals[totals.sum(axis="columns")>10000]

percent=totals.div(totals.sum(axis="columns"), axis="index")
percent.head(10)
```

### Relative percentage of total donations by state for each candidate

![image](https://github.com/EduardoJMR/Federal-Election-project/blob/master/images/Capture12.JPG)







