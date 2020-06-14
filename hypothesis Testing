# Use this dictionary to map state names to two letter acronyms
states = {'OH': 'Ohio', 'KY': 'Kentucky', 'AS': 'American Samoa', 'NV': 'Nevada', 'WY': 'Wyoming', 'NA': 'National', 'AL': 'Alabama', 'MD': 'Maryland', 'AK': 'Alaska', 'UT': 'Utah', 'OR': 'Oregon', 'MT': 'Montana', 'IL': 'Illinois', 'TN': 'Tennessee', 'DC': 'District of Columbia', 'VT': 'Vermont', 'ID': 'Idaho', 'AR': 'Arkansas', 'ME': 'Maine', 'WA': 'Washington', 'HI': 'Hawaii', 'WI': 'Wisconsin', 'MI': 'Michigan', 'IN': 'Indiana', 'NJ': 'New Jersey', 'AZ': 'Arizona', 'GU': 'Guam', 'MS': 'Mississippi', 'PR': 'Puerto Rico', 'NC': 'North Carolina', 'TX': 'Texas', 'SD': 'South Dakota', 'MP': 'Northern Mariana Islands', 'IA': 'Iowa', 'MO': 'Missouri', 'CT': 'Connecticut', 'WV': 'West Virginia', 'SC': 'South Carolina', 'LA': 'Louisiana', 'KS': 'Kansas', 'NY': 'New York', 'NE': 'Nebraska', 'OK': 'Oklahoma', 'FL': 'Florida', 'CA': 'California', 'CO': 'Colorado', 'PA': 'Pennsylvania', 'DE': 'Delaware', 'NM': 'New Mexico', 'RI': 'Rhode Island', 'MN': 'Minnesota', 'VI': 'Virgin Islands', 'NH': 'New Hampshire', 'MA': 'Massachusetts', 'GA': 'Georgia', 'ND': 'North Dakota', 'VA': 'Virginia'}
import pandas as pd
import numpy as np
from scipy.stats import ttest_ind
def get_list_of_university_towns():
    '''Returns a DataFrame of towns and the states they are in from the 
    university_towns.txt list. The format of the DataFrame should be:
    DataFrame( [ ["Michigan", "Ann Arbor"], ["Michigan", "Yipsilanti"] ], 
    columns=["State", "RegionName"]  )
    
    The following cleaning needs to be done:

    1. For "State", removing characters from "[" to the end.
    2. For "RegionName", when applicable, removing every character from " (" to the end.
    3. Depending on how you read the data, you may need to remove newline character '\n'. '''
    
    # Have to specify entries are separated by new line. Setting header = None stops the first row entry from becoming the column name.
    uni_towns = pd.read_csv('university_towns.txt',sep='\n', header=None, names=['RegionName'])
        
    # Enter state name in column 'State' in the same row as 'RegionName'. Set everything else to NaN.
    uni_towns['State'] = np.where(uni_towns['RegionName'].str.contains('edit'),uni_towns['RegionName'],np.NaN)
        
    uni_towns['State'].fillna(method='ffill',inplace=True) # Forward fill state names to replace NaN values.
    
    uni_towns = uni_towns[['State','RegionName']] # Rearrange columns to required order
    
    # Use '(' and '[' split for both columns even though it doesn't ask you to! That's the main bug.
    # expand = True splits the string into different columns.
    for col in uni_towns:
        uni_towns[col] = uni_towns[col].str.split('(',expand=True)[0].str.split('[', expand=True)[0].str.rstrip()
        
    # Remove rows where State and RegionName have the same entry:
    uni_towns = uni_towns[uni_towns['State'] != uni_towns['RegionName']]    
    
    return uni_towns
    
    
get_list_of_university_towns()

def read_gdp():
    
    GDP = pd.read_excel('gdplev.xls', skiprows=4) # Drop irrelevant rows.
    
    GDP = GDP.drop(GDP.columns[[0,1,2,3,5,7]],axis=1) # Remove unnecessary data.
    
    # Setting the first row to header:
    new_header = GDP.iloc[0]
    GDP = GDP[3:]
    GDP.columns = new_header
    
    # More cleaning:
    GDP = GDP.reset_index(drop=True)
    GDP.columns = ['Quarter','GDP']
    
    # Remove data before 2000 Q1:
    GDP = GDP.drop(GDP.index[0:212])
    #return GDP.index[GDP['Quarter'] == '2000q1'] # Returns index = 212 for 2000q1.
    
    GDP = GDP.reset_index(drop=True)
    
    GDP['GDP Diff'] = GDP['GDP'].diff() # Finds the difference between successive row entries in the column 'GDP'.
    
    return GDP
def get_recession_start():
    '''Returns the year and quarter of the recession start time as a 
    string value in a format such as 2005q3'''
    
    GDP = read_gdp()
    
    # Finds all the quarters where there is a decline:
    GDP_dec = GDP.where(GDP['GDP Diff']<0)
    GDP_dec = GDP_dec.dropna()
    
    # Find the first quarter with a successive decline:
    GDP_dec['Index'] = GDP_dec.index # Get index values into a column to use diff().
    GDP_dec['Index Diff'] = GDP_dec['Index'].diff() # Find the difference for index values.
    min_index = GDP_dec['Index Diff'].idxmin() # Find the FIRST quarter where index difference is 1. idxmin() gives the first occurence of the minimum value.
    
    return GDP['Quarter'].iloc[min_index-1] # You want the first quarter of the 2 successive quarters with a decline.

get_recession_start()


def get_recession_end():
    '''Returns the year and quarter of the recession end time as a 
    string value in a format such as 2005q3'''
    
    GDP = read_gdp()
    
    # Finds all the quarters where there is a rise:
    GDP_rise = GDP.where(GDP['GDP Diff']>0)
    GDP_rise = GDP_rise.dropna()
    
    # Find the first quarter after the recession starts where there is a successive rise:
    GDP_rise['Index'] = GDP_rise.index
    GDP_rise['Index Diff'] = GDP_rise['Index'].diff()
    max_index = GDP_rise['Index Diff'].idxmax()
    
    # Any quarter with an index difference of more than 3 has had at least 2 successive declining quarters before it!
    
    # Recession ends at the second quarter of growth. Therefore we use (max_index+1).
    return GDP['Quarter'].iloc[max_index+1]
       
get_recession_end()

def get_recession_bottom():
    '''Returns the year and quarter of the recession bottom time as a 
    string value in a format such as 2005q3'''
    
    GDP = read_gdp()
    
    # Get indices of recession start and end quarters:
    start_index = GDP.loc[GDP['Quarter'] == get_recession_start()].index.astype(int)[0]
    end_index = GDP.loc[GDP['Quarter'] == get_recession_end()].index.astype(int)[0]
    
    # Limit GDP to recession range. Use (end_index + 1) because the end index isn't included in the range.
    GDP = GDP.iloc[start_index:end_index+1]
    
    # Return the quarter of the entry where GDP = minimum GDP:
    return GDP['Quarter'][GDP.loc[GDP['GDP'] == GDP['GDP'].min()].index.astype(float)[0]]

get_recession_bottom()


# Change year and month column header to year and quarter:

def change_to_quarter(date: str):
    date = date.split('-')
    month = int(date[1])
    quarter = int((month - 1) / 3) + 1
    return date[0] + 'q' + str(quarter)
    
    def convert_housing_data_to_quarters():
    '''Converts the housing data to quarters and returns it as mean 
    values in a dataframe. This dataframe should be a dataframe with
    columns for 2000q1 through 2016q3, and should have a multi-index
    in the shape of ["State","RegionName"].
    
    Note: Quarters are defined in the assignment description, they are
    not arbitrary three month periods.
    
    The resulting dataframe should have 67 columns, and 10,730 rows.
    '''
    
    all_homes = pd.read_csv('City_Zhvi_AllHomes.csv')
    
    # Drop columns with unnecessary data:
    start_rem = all_homes.columns.get_loc('1996-04')
    end_rem = all_homes.columns.get_loc('2000-01')
    all_homes = all_homes.drop(all_homes.columns[start_rem:end_rem],axis=1) 
    
    # Double square brackets if you're giving specific colums to remove, single square brackets if it's a column range.
    
    # Removing more unnecessary columns:
    all_homes = all_homes.drop(all_homes.columns[[0,3,4,5]],axis=1)
    
    # Map state short forms with given dictionary:
    all_homes['State'] = all_homes['State'].map(states)
    
    # Switch State and RegionName columns for multiindex:
    columnsName = list(all_homes.columns)
    S, R = columnsName.index('State'), columnsName.index('RegionName')
    columnsName[S], columnsName[R] = columnsName[R],columnsName[S]
    all_homes = all_homes[columnsName]
    
    # Sorts and groups by index:
    all_homes = all_homes.set_index(['State','RegionName']).sort_index()
    
    # Group by user defined function (above) which changes given dates to year + quarter. Axis = 1 specifies you're passing column names to the function.
    all_homes = all_homes.groupby(change_to_quarter, axis=1).mean() # Find mean over the months in a quarter.
    
    return all_homes

convert_housing_data_to_quarters()

#Test:
#convert_housing_data_to_quarters().loc[[('Ohio','Akron'),('Ohio','Dayton')]].loc[:,['2010q3','2015q2','2016q4']]


def run_ttest():
    '''First creates new data showing the decline or growth of housing prices
    between the recession start and the recession bottom. Then runs a ttest
    comparing the university town values to the non-university towns values, 
    return whether the alternative hypothesis (that the two groups are the same)
    is true or not as well as the p-value of the confidence. 
    
    Return the tuple (different, p, better) where different=True if the t-test is
    True at a p<0.01 (we reject the null hypothesis), or different=False if 
    otherwise (we cannot reject the null hypothesis). The variable p should
    be equal to the exact p value returned from scipy.stats.ttest_ind(). The
    value for better should be either "university town" or "non-university town"
    depending on which has a lower mean price ratio (which is equivilent to a
    reduced market loss).'''
    
    df = convert_housing_data_to_quarters()
    
    # Start position is the quarter BEFORE the recession starts!
    before_rec = (df.columns.get_loc(get_recession_start())-1)
    rec_bottom = df.columns.get_loc(get_recession_bottom())
    
    uni = get_list_of_university_towns().set_index(['State', 'RegionName'])
    
    # Turn the divided values into a DataFrame!
    df = np.divide(df.ix[:,before_rec],df.ix[:,rec_bottom]).to_frame().dropna()
    
    # Merge university and GDP data.
    uni_df = df.merge(uni, right_index=True, left_index=True, how='inner')
    
    # Drop the indices of uni towns to get data only for non uni towns.
    nonuni_df = df.drop(uni_df.index)
    
    # A t-test is commonly used to determine whether the mean of a population significantly
    # differs from a specific value (called the hypothesized mean) or from the mean of another population.
    p_value = ttest_ind(uni_df.values, nonuni_df.values).pvalue
    
    if p_value < 0.01:
        different=True
    else:
        different=False
        
    # Better depending on which one is LOWER! Remember prices go up during a recession so lower is better.
    if uni_df.mean().values < nonuni_df.mean().values:
        better='university town'
    else:
        better='non-university town'

    return (different, p_value[0], better)
    
run_ttest()

