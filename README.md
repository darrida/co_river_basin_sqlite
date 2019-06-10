# SQLite Script for Colorodo River Basin Cooperation and Conflict Spreadsheet
This is a simple Python script I created to insert the Colorodo River Basin Cooperation and Conflict Database into a SQLite database. 

The reposity is associated with three files:
- co_river_basin_insert.ipynb -- Jupyter Notebook file
- co_river_basin_insert.py -- Python file
- Final_WWIS_UC_Event_Database.xls -- source of inserted data

The Jupyter Notebook and Python files are located here; see the Source section below for the location of the data file.

## Source
Project site that the data originates from: https://transboundarywaters.science.oregonstate.edu/content/collection-research-and-datasets-colorado-river-basin

**Direction download link**: https://transboundarywaters.science.oregonstate.edu/sites/transboundarywaters.science.oregonstate.edu/files/Database/ResearchProjects/Colorado/Final_WWIS_UC_Event_Database.xls

**Citation**: “Product of the Transboundary Freshwater Dispute Database, College of Earth, Ocean, and Atmospheric Sciences, Oregon State University.  Additional information about the TFDD can be found at: http://transboundarywaters.science.oregonstate.edu.” 

Side note: I am not associated with the creation of that project or Oregon State University.

## SQL Script for SQLite table:
```
CREATE TABLE CO_River_Basin (
    Event            DECIMAL (10)   PRIMARY KEY
                                    UNIQUE
                                    NOT NULL,
    SearchSource     VARCHAR (100),
    Newspaper        VARCHAR (200),
    ArticleCitation  VARCHAR (1000),
    ArticleTitle     VARCHAR (1000) NOT NULL,
    File             VARCHAR (100),
    ReportDate       DATE,
    ReportYear       DECIMAL (10),
    Eventdate        VARCHAR (100),
    EventYear        VARCHAR (100),
    Basin            VARCHAR (100),
    LocatorWaterBody VARCHAR (200),
    IssueType        VARCHAR (500),
    EventSummary     TEXT (5000),
    Stakeholders     VARCHAR (500),
    IntensityValue   VARCHAR (6),
    WaterTitle       VARCHAR (200),
    Comments         VARCHAR (5000) 
);
```
## Python Script
```
import pandas as pd
import sqlalchemy
import sqlite3
from IPython.core.debugger import set_trace
import sys

df_co_river_basin = pd.read_excel('Final_WWIS_UC_Event_Database.xls')

# USED TO VISUALIZE CURRENT COLUMN NAMES
#for col in df_co_river_basin.columns: 
#    print(col)

# PANDAS FUNCTION THAT RENAMES DATAFRAME COLUMNS
df_co_river_basin.rename(columns={'Search Source':'SearchSource',
                                  'Article Citation': 'ArticleCitation',
                                  'Article Title': 'ArticleTitle',
                                  'Report Date':'ReportDate',
                                  'Report Year':'ReportYear',
                                  'Event date':'Eventdate',
                                  'Event Year':'EventYear',
                                  'Locater/ Water Body':'LocatorWaterBody',
                                  'Issue Type':'IssueType',
                                  'Event Summary':'EventSummary',
                                  'Intensity Value':'IntensityValue',
                                  'Water Title':'WaterTitle'}, inplace=True)

# USED TO VISUALIZE COLUMN NAME CHANGES
#print('\nFIXED NAMES:')    
#for col in df_co_river_basin.columns: 
#    print(col)

# USED TO SEE SAMPLE OF THE DATAFRAME
#df_co_river_basin

# GENERAL FORMATTING ESCAPE LINE I INCLUDE IN MOST DATAFRAMES TO INSTERT INTO DATABASES
df_co_river_basin = df_co_river_basin.applymap(lambda x: x.encode('unicode_escape').decode('utf-8') if isinstance(x, str) else x)

# REMOVE EMPTY ROWS
df_co_river_basin = df_co_river_basin[pd.notnull(df_co_river_basin['Event'])]

# CONNECTION TO LOCAL SQLITE DATABASE
conn = sqlite3.connect('C:\\sqlite\\databases\\waterdb\\waterdb.db')
c = conn.cursor()

# LOOP THROUGH DATAFRAME AND INSERT INTO SQLITE DB
# Note: this isn't very clean - I worked with the values section until it took everything. If I did this again, I'd probably run the 'replace' procedure across the dataframe futher above.
for i, row in enumerate(df_co_river_basin.itertuples(), 1):
    testEvent = pd.read_sql("SELECT Event FROM CO_River_Basin WHERE Event = " + str(row.Event), conn)
    if testEvent.empty == True:
        try:
            c.execute("""INSERT INTO CO_River_Basin
                        (Event,
                         SearchSource,
                         Newspaper,
                         ArticleCitation,
                         ArticleTitle,
                         File,
                         ReportDate,
                         ReportYear,
                         Eventdate,
                         EventYear,
                         Basin,
                         LocatorWaterBody,
                         IssueType,
                         EventSummary,
                         Stakeholders,
                         IntensityValue,
                         WaterTitle,
                         Comments)
                    VALUES
                        ('""" + str(row.Event) + """',
                         '""" + str(row.SearchSource).replace("'","\u2018") + """',
                         '""" + str(row.Newspaper).replace("'","\u2018") + """',
                         '""" + str(row.ArticleCitation).replace("'","\u2018") + """',
                         '""" + str(row.ArticleTitle).replace("'","\u2018") + """',
                         '""" + str(row.File) + """',
                         '""" + str(str(row.ReportDate)[0:10]) + """',
                         '""" + str(row.ReportYear).replace("'","\u2018") + """',
                         '""" + str(row.Eventdate).replace("'","\u2018") + """',
                         '""" + str(row.EventYear).replace("'","\u2018") + """',
                         '""" + str(row.Basin).replace("'","\u2018") + """',
                         '""" + str(row.LocatorWaterBody).replace("'","\u2018") + """',
                         '""" + str(row.IssueType).replace("'","\u2018") + """',
                         '""" + str(row.EventSummary).replace("'","\u2018") + """',
                         '""" + str(row.Stakeholders).replace("'","\u2018") + """',
                         '""" + str(row.IntensityValue) + """',
                         '""" + str(row.WaterTitle).replace("'","\u2018") + """',
                         '""" + str(row.Comments) + """')""")
            sys.stdout.write('\r' + 'Processed: ' + str(row.Event))
            conn.commit()
        except Exception as e:
            print(row)
            print('ERROR: ' + repr(e))
            set_trace()
        sys.stdout.write('\r' + 'Processed: ' + str(row.Event))
    else:
        sys.stdout.write('\r' + 'Processed: ' + str(row.Event))        
sys.stdout.write('\r' + 'Processed: ' + str(row.Event) + "| FINISHED")
```
