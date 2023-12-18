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

Becuase most of the data came from Wikipedia, most of the data already existed in tabular data independently online as well. For that reason, the main challenge here was reading html tables, cleaning data, joining tables, cleaning data again, and then repeating iteratively until completion. 

## Variables