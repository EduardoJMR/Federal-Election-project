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

### Transforming the data

##### It looks like there is missed the party of every candidate in the dataset, so to do further analysis with this information, let us start working on this, then add this information as a column.

```python
unique_cands = fec["cand_nm"].unique()
```

```python
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
```
```python
pd.Series(parties, index=['Bachman, Michelle', 'Cain, Herman', 'Gingrich, Newt', 'Huntman, Jon', 'McCotter, Thaddeus G', 'Obama, Barack', 'Paul, Ron', 'Pawlenty, Timothy', 'Perry, Rick', "Roemer, Charles E. 'Buddy' III", 'Romrey, Mitt', 'Santorum, Rick'])
```
```python
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



