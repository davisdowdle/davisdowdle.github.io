---
layout: post
title:  Worldwide GDP Per Capita Data Acquisition
author: Davis Dowdle
description: Here's how I collected the data for analyzing international economic data.
image: "https://upload.wikimedia.org/wikipedia/commons/e/ee/Map_of_countries_by_GDP_%28nominal%29_per_capita_in_2023.svg"
--- 

Some things never change, like the fact that humans are always competing. Specifically, for millenia, nation has fought nation to become the economic superpower of the world; from mercantilism to conquest to protectionism and so on, one of the greatest human competitions has been to create the most coveted economy. It's no different today. Economists, politicians, and voters alike are all interested in investing in their economy to make it the symbol of their country's superiority. But how do countries achieve this? How does a country create wealth for its citizens? What factors contribute to the overall evonomic health of a country? I wanted to see what patterns I could find, so I did the below web scraping to analyze.

## General Process

A common token of economic success in the international conversation is GDP per capita, especially that adjusted for Purchasing Power Parity (PPP). Finding this data for an adequate selection of countries/territories proved to be fairly difficult. In fact, finding robust international data for any variable was difficult without using Wikipedia. For that reason, I give a special thank you to Wikipedia for supplying the majority of the data in this endeavor. 

Becuase the majority of the data came from Wikipedia, most of it already existed in tabular data independently online as well. For that reason, the main challenge here was reading html tables, cleaning data, joining tables, cleaning data again, and then repeating iteratively until completion. 

The only necessary packages for this method are `pandas` and `re`. Below, the package `pandas` is represented as `pd`.

## Variables

The variables I collected for each relevant country/territory were name, parent country (if applicable for territories), basic region according to the United Nations, primary currency, real GDP per capita (PPP), total population, total land area (sq mi), population density derived from the previous two fields, and trade ratio derived from dividing exports by imports. 

# Step by Step

## Real GDP per Capita

As said, I acquired the GDP data from Wikipedia, who got the data from the CIA. The associated table can be viewed at the url given in the below code excerpt, which is the chunk for reading and cleaning the table. Notice that the table also contained the UN region of the country.

```python
gdpurl = "https://en.wikipedia.org/wiki/List_of_countries_by_GDP_(PPP)_per_capita"
gdptables = pd.read_html(gdpurl)
gdptable = gdptables[1]
gdptable.columns = gdptable.columns.droplevel(0) #remove second-level column index
gdptable = gdptable.iloc[:,[0, 6, 1]].sort_values('Estimate', ascending = False) #extract valuable columns and sort descending
gdptable['Country/Territory'] = [next.replace('[n 1]', '').strip('*').upper() for next in \
    [country.encode('ascii', 'ignore').decode('unicode_escape') for country in gdptable['Country/Territory']]] #remove ascii keys and other characters from country names
gdptable = gdptable.reset_index(drop = True) 
gdptable.loc[55, 'Country/Territory'] = 'US VIRGIN ISLANDS' #hard code discrepant country names for future joining
gdptable.loc[57, 'Country/Territory'] = 'SINT MAARTEN'
gdptable.loc[89, 'Country/Territory'] = 'CURACAO'
gdptable.loc[93, 'Country/Territory'] = 'SAINT MARTIN'
gdptable.loc[184, 'Country/Territory'] = 'SAO TOME AND PRINCIPE'
```

First, I copied the url and then used a simple `pd.read_html` to extract the second table from the html. Wikipedia tables are known for being pretty messy to deal with because of annotations, footnotes, ascii keys, irregular tables structures, name inconsistencies, and so on. In this case, the table's column format introduced a multi-indexed structure to the data frame, which had to be removed. Additionally, some ascii keys and various annotation symbols had to be omitted. Finally, several country/territory names needed to be amended for naming consistency down the road. 

This tedious cleaning yielded the table below with the "Estimate" column being the CIA's approximation of the country's 2019 Real GDP per Capita. 

![gdptable]({{site.url}}.{{site.baseurl}}/assets/images/gdptable.png)

If you think this step had some annoying wrangling and cleaning, buckle up. It only got worse down the road.

## Population, Geographic Area, Population Density

Next, I acquired the total population, land surface area, and the population density for each country/territory through Wikipedia. This table also helpfully supplied the sovereign state of each dependency. Observe the excerpt below.

```python
popurl = "https://en.wikipedia.org/wiki/List_of_countries_and_dependencies_by_population_density"
poptables = pd.read_html(popurl)
poptable = poptables[0]
poptable = poptable.iloc[:, [1, 4, 6, 3]] #extract valuable columns
poptable['Country/Territory'] = [country.split('(')[0].strip().upper() for country in poptable['Country or dependency']] #extract country name
parents = [] #define parents list appending country name if not a subsidiary and parent country name if a subsidiary (evident by parentheses following entity name)
for country in poptable['Country or dependency']:
    if bool(re.search('\(', country)):
        parent = country.split('(')[1].strip(')').upper()
    else:
        parent = country.split('(')[0].strip().upper()
    parents.append(parent)
poptable['Parent'] = parents #define parent country columns
poptable = poptable.iloc[:, [4, 5, 1, 2, 3]] #extract and rearrange columns
poptable.loc[30, 'Country/Territory'] = 'CURACAO' #hard code discrepant country names for future joining
poptable.loc[60, 'Country/Territory'] = 'SAO TOME AND PRINCIPE'
```

Everything was pretty similar to the former table except for one thing--obtaining the parent country of a dependency. In the table, those entities were following in parentheses by its governing state while independent countries were standalone (disregarding footnotes and annotations). As such, in addition to the list comprehension step employed for the GDP table to clean the entity name, I added a for loop containing an if statement respecting the independence status of the entity. If parentheses were detected, the information in the parentheses was extracted as the parent country and appended to a list of parents. Otherwise, the country's name itself was used to indicate that the country was independent. 

**Wikipedia is regularly updated. In between my data collection and writing this post, this Wikipedia page was revised rendering the above code invalid. For that reason, I don't have a picture available of the final population data. The excerpt above should be a good framework for you to correctly wrangle the data post-revision.**

## Currency

I obtained adequate currency information for each necessary country and territory from the IBAN website. 

```python
currurl = "https://www.iban.com/currency-codes"
currtables = pd.read_html(currurl)
currtable = currtables[0]
currtable['Country'] = [country.split('(')[0].strip() for country in currtable['Country']] #fix country names
currtable = currtable.iloc[:, [0, 2]] #extract country names and currency codes
currtable.loc[253, 'Country'] = 'UNITED STATES' #hard code discrepant country names for future joining
currtable.loc[35, 'Country'] = 'BRUNEI'
currtable.loc[251, 'Country'] = 'UNITED KINGDOM'
currtable.loc[127, 'Country'] = 'SOUTH KOREA'
currtable.loc[263, 'Country'] = 'US VIRGIN ISLANDS'
currtable.loc[262, 'Country'] = 'BRITISH VIRGIN ISLANDS'
currtable.loc[196, 'Country'] = 'RUSSIA'
currtable.loc[62, 'Country'] = 'CURACAO'
currtable.loc[141, 'Country'] = 'NORTH MACEDONIA'
currtable.loc[261, 'Country'] = 'VIETNAM'
currtable.loc[228, 'Country'] = 'ESWATINI'
currtable.loc[130, 'Country'] = 'LAOS'
currtable.loc[39, 'Country'] = 'CAPE VERDE'
currtable.loc[58, 'Country'] = 'IVORY COAST'
currtable.loc[238, 'Country'] = 'EAST TIMOR'
currtable.loc[233, 'Country'] = 'SYRIA'
currtable.loc[236, 'Country'] = 'TANZANIA'
currtable.loc[126, 'Country'] = 'NORTH KOREA'
currtable.loc[54, 'Country'] = 'DR CONGO'
currtable.loc[len(currtable.index)] = ['MACAU', 'MOP'] #hard code country/territories not found in iban table
currtable.loc[len(currtable.index)] = ['KOSOVO', 'EUR']
currtable.loc[len(currtable.index)] = ['PALESTINE', 'ILS']
```

As evident in the above code, aside from a little string manipulation, much of the work in this step was hard-coding country names to correct discrepancies for joining the tables later. Some entities were also not found in the IBAN table, so those had to be hard-coded as well. 

Below is the currency table post-cleaning. 

![currtable]({{site.url}}.{{site.baseurl}}/assets/images/currtable.png)

## Exports / Imports

The CIA not only publishes GDP data for each country, but it also published exports and imports data. I used the 2019 round of data as much as I could. Below is the code for both tables.

```python
expurl = "https://www.cia.gov/the-world-factbook/field/exports/country-comparison/"
exptables = pd.read_html(expurl)
exptable = exptables[0]
exptable['Country'] = [country.upper() for country in exptable['Country']] #fix country names to uppercase for joining
exptable['Exports'] = [int(country.strip('$').replace(',', '')) for country in exptable['Unnamed: 2']] #extract exports as integer
exptable = exptable.iloc[:, [1, 4]] #extract country name and exports
```

```python
impurl = "https://www.cia.gov/the-world-factbook/field/imports/country-comparison/"
imptables = pd.read_html(impurl)
imptable = imptables[0]
imptable['Country'] = [country.upper() for country in imptable['Country']] #fix country names to uppercase for joining
imptable['Imports'] = [int(country.strip('$').replace(',', '')) for country in imptable['Unnamed: 2']] #extract imports as integer
imptable = imptable.iloc[:, [1, 4]] #extract country name and exports
```

Again, most of the cleaning in this phase was just manipulating strings, including making countries totally upper case and getting rid of dollar signs and commas so they could be converted to integers instead of strings. 

After priming these two tables, I joined the tables using the code below.

```python
trade = exptable.merge(imptable, how = 'outer', left_on = 'Country', right_on = 'Country') #join exports and imports, hardcoding Liechtenstein below using data from same source just one year earlier
trade.loc[142, 'Exports'] = 3774000000 #https://www.cia.gov/the-world-factbook/countries/liechtenstein/#economy
trade.loc[142, 'Imports'] = 2230000000 #https://www.cia.gov/the-world-factbook/countries/liechtenstein/#economy
trade['Ratio'] = trade['Exports'] / trade['Imports'] #define trade ratio columns
trade.loc[197, 'Country'] = 'FALKLAND ISLANDS' #hard code country names
trade.loc[7, 'Country'] = 'SOUTH KOREA'
trade.loc[33, 'Country'] = 'CZECH REPUBLIC'
trade.loc[162, 'Country'] = 'US VIRGIN ISLANDS'
trade.loc[28, 'Country'] = 'TURKEY'
trade.loc[143, 'Country'] = 'BAHAMAS'
trade.loc[215, 'Country'] = 'SAINT HELENA, ASCENSION AND TRISTAN DA CUNHA'
trade.loc[188, 'Country'] = 'CAPE VERDE'
trade.loc[93, 'Country'] = 'IVORY COAST'
trade.loc[153, 'Country'] = 'EAST TIMOR'
trade.loc[85, 'Country'] = 'MYANMAR'
trade.loc[202, 'Country'] = 'MICRONESIA'
trade.loc[130, 'Country'] = 'CONGO'
trade.loc[205, 'Country'] = 'GAMBIA'
trade.loc[199, 'Country'] = 'NORTH KOREA'
trade.loc[82, 'Country'] = 'DR CONGO' #hard code exports/imports for missing countries
trade.loc[len(trade.index)] = ['ISLE OF MAN', 432000000, 922000000, 432000000/922000000] #https://assets.publishing.service.gov.uk/media/653fbb156de3b9000da7a609/isle-of-man-trade-and-investment-factsheet-2023-11-01.pdf
trade.loc[len(trade.index)] = ['JERSEY', 3900000000, 4300000000, 3900000000/4300000000] #https://assets.publishing.service.gov.uk/media/653fbe9246532b000d67f545/jersey-trade-and-investment-factsheet-2023-11-01.pdf
trade.loc[len(trade.index)] = ['GUERNSEY', 953000000, 3100000000, 953000000/3100000000] #https://assets.publishing.service.gov.uk/media/653fba4746532b001467f52f/guernsey-trade-and-investment-factsheet-2023-11-01.pdf
trade.loc[len(trade.index)] = ['SAINT MARTIN', 23700000, 529000000, 23700000/529000000] #https://oec.world/en/profile/country/maf
trade.loc[len(trade.index)] = ['PALESTINE', 1450000000, 6940000000, 1450000000/6940000000] #https://oec.world/en/profile/country/pse
```

In this step, a lot of hard-coding had to take place to, again, correct inconsistent naming and incorporate missing entities. The missing entities' data had to be looked up individually at the corresponding links given above. 

Notice also that I generated the trade ratio column, the quotient of the exports and the imports. 

Here is what the trade data looked like following the execution of the above code chunks. 

![tradetable]({{site.url}}.{{site.baseurl}}/assets/images/tradetable.png)

## Merges

The last step was to merge all the tables and finalize with some cleaning.

First, the GDP, population, and currency tables were joined with the excerpt below.

```python
inter = gdptable.merge(poptable, how = 'inner', left_on = 'Country/Territory', right_on = 'Country/Territory') #join gdp and pop
inter = inter[inter['Country/Territory'].duplicated() == False] #remove repeat countries
inter = inter.merge(currtable, how = 'left', left_on = 'Country/Territory', right_on = 'Country') #join gdp, pop, and curr
```

The only abnormality in this step was that duplicate rows had to be removed after merging the GDP and population tables.

After the former three tables merged, I merged the resultant dataframe with the trade dataframe. 

```python
final = inter.merge(trade, how = 'left', left_on = 'Country/Territory', right_on = 'Country') #merge all
final = final.iloc[:, [0, 3, 2, 8, 1, 4, 5, 6, 12]] #rearrange columns
final.columns = ['Entity', 'Parent', 'Region', 'Currency', 'GDP', 'Population', 'Area', 'Density', 'Ratio'] #rename columns
final.iloc[final[final['Parent'] == 'MICRONESIA'].index, 0] = 'FEDERATED STATES OF MICRONESIA' #fix parent country names to correct/consistent names
final.iloc[final[final['Parent'] == 'CONGO'].index, 0] = 'REPUBLIC OF THE CONGO'
final.iloc[final[final['Parent'] == 'DR CONGO'].index, 0] = 'DEMOCRATIC REPUBLIC OF THE CONGO'
final.iloc[final[final['Parent'] == 'UK'].index, 1] = 'UNITED KINGDOM'
final.iloc[final[final['Parent'] == 'US'].index, 1] = 'UNITED STATES'
final.iloc[final[final['Parent'] == 'NL'].index, 1] = 'NETHERLANDS'
final.iloc[final[final['Parent'] == 'NZ'].index, 1] = 'NEW ZEALAND'
final.iloc[final[final['Parent'] == 'COOK ISLANDS'].index, 1] = 'NEW ZEALAND'
final.iloc[final[final['Parent'] == 'NIUE'].index, 1] = 'NEW ZEALAND'
final.iloc[final[final['Parent'] == 'SÃO TOMÉ AND PRÍNCIPE'].index, 1] = 'SAO TOME AND PRINCIPE'
final.iloc[final[final['Parent'] == 'CONGO'].index, 1] = 'REPUBLIC OF THE CONGO'
final.iloc[final[final['Parent'] == 'DR CONGO'].index, 1] = 'DEMOCRATIC REPUBLIC OF THE CONGO'
final.iloc[final[final['Parent'] == 'METROPOLITAN'].index, 1] = 'FRANCE'
final.iloc[final[final['Parent'] == 'MICRONESIA'].index, 1] = 'FEDERATED STATES OF MICRONESIA'
final.iloc[final[final['Parent'] == 'EXCLUDING ANTARCTICA'].index, 1] = 'WORLD'
final.drop(index = [8, 10, 75, 81, 86, 100, 117, 136, 147, 155, 158, 166, 170, 210, 219], inplace = True) #remove country indexes duplicated because of currencies (most official currency favored)
final = final.reset_index(drop = True)
```

Some final cleaning at the end was necessary to reconcile all inconsistencies. I renamed coolumns, hard-coded some corrections to discrepancies in country and parent country names, and selected only the most prominent currency for countries that accepted multiple. 

Voila. See the beautiful table directly below. The final product.

![finaltable]({{site.url}}.{{site.baseurl}}/assets/images/finaltable.png)