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

