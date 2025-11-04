### This code cleans two .csv's 

### The CSV's are merged based on county and state names 

### Steps: 

## Clean Electric vehicle population size by county csv 

```bash
ev_data=read.csv(file.choose()) #read in file
```
### only include 2023 years because the continuum codes were sampled 2023 
ev_data_filtered_date = ev_data[grepl("2023", ev_data$Date), ]

### only selecting Date, County, State, VPU, PEV
ev_data_filtered_date.sub=ev_data_filtered_date[,c(1, 2,3,4,10)] #create data subset with all rows but only the columns of interest
head(ev_data_filtered_date.sub) #confirm correct columns were selected

### rename sub-setted columns 
names(ev_data_filtered_date.sub) = c("Date", "County", "State", "VPU", "PEV")
head(ev_data_filtered_date.sub) # confirm renaming worked 

### create data subset without rows with NA in any cell
ev_data2=na.omit(ev_data_filtered_date.sub) 
nrow(ev_data2) #view new number of rows
head(ev_data2)

### create data subset without rows with empty County and State fields
### only keep the row if state and county are both filled, otherwise drop 
ev_data3=ev_data2[ev_data2$County!="" | ev_data2$State!="",] 
nrow(ev_data3) #view new number of rows

### turn VPU (vehicle primary use) into cat. (Categorical- Passenger (0) or Truck (1))
ev_data3$VPU = ifelse(ev_data3$VPU == "Passenger", 0,
                      ifelse(ev_data3$VPU == "Truck", 1, NA))
### verify that we stored the values as numeric 
is.numeric(ev_data3$VPU)

## Clean rural-urban csv 
cont_codes_data=read.csv(file.choose()) #read in file
head(cont_codes_data) #view first 6 rows of data to see variable names and data formatting
nrow(cont_codes_data) #view number of rows
ncol(cont_codes_data) #view number of columns

### only selecting State, County Name, Value (metro/nonmetro) columns
cont_codes_data.sub=cont_codes_data[,c(2, 3, 5)] #create data subset with all rows but only the columns of interest
head(cont_codes_data.sub) #confirm correct columns were selected

### rename columns for brevity 
names(cont_codes_data.sub) = c("State", "County", "Metro_Status")
head(cont_codes_data.sub) # confirm names are correct 

cont_codes_data2=na.omit(cont_codes_data.sub) 
nrow(cont_codes_data2) #view number of rows

### turn Metro_Status into cat. (Categorical- Metro (1) or Nonmetro (0))
cont_codes_data2$Metro_Status = ifelse(grepl("^Metro", cont_codes_data2$Metro_Status), 1,
                      ifelse(grepl("^Nonmetro", cont_codes_data2$Metro_Status), 0, NA))

is.numeric(cont_codes_data2$Metro_Status)

### remove all N/A sections 
cont_codes_data3 = na.omit(cont_codes_data2)
### reindex row numbers after wiping rows with NA 
rownames(cont_codes_data3) = NULL
### view cleaned data 
nrow(cont_codes_data3)
head(cont_codes_data3)

### change encoding to interpret accents above county names for county cleanup 
cont_codes_data3$County = iconv(cont_codes_data3$County, from = "", to = "UTF-8")

### County Cleanup: 
### remove trailing "county" string from listed counties 
cont_codes_data3$County=gsub(" County$", "", cont_codes_data3$County) 
head(cont_codes_data3)


## Merge two .csv's into one 

### Merge two csv's based on State and County

### keeps all rows from ev_data3, & adds rows from cont_codes
merged_data = merge(ev_data3, cont_codes_data3,
                    by = c("State", "County"),
                    all.x = TRUE) ### left join. 


### remove all N/A sections 
omitted_merged_data = na.omit(merged_data)

### reindex row numbers after wiping rows with NA 
rownames(omitted_merged_data) = NULL


### check for any NA values for a final check 
anyNA(omitted_merged_data)

