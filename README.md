# utl-combining-every-two-rows-into-row-in-R-WPS-and-SAS
Combining every two rows into one row in R WPS and SAS
    %let pgm=utl-combining-every-two-rows-into-row-in-R-WPS-and-SAS;

    Combining every two rows into one row in R WPS and SAS

    Related question
    https://tinyurl.com/3cudafyd
    https://stackoverflow.com/questions/73805451/combine-every-two-rows-of-data-in-r

    SOAPBOX ON

       R and Python complement SAS/WPS, but no way are they replacements.
       The biggest issues with R and Python is how they silently switch data structures and data types.
       It can be extremely difficult to convert the myriad of data structures to something you
       can use in another language.
       SQL is a partial solution to this issue.
       Note the double 'as.data.frame' below.

       The only additional functionality SAS needs is 128bit IEEE floats for interoperability with all
       languages.

    SOAPBOX OFF

    /*
     _                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    d:/txt/twoRow.txt

    100 33 100000 5000
    100 61 180
    200 55 90000 4000
    200 71 150
    300 66 81000 3500
    300 78 210

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    Up to 40 obs from WANT total obs=3 21SEP2022:16:16:12

    Obs     ID    AGE    INCOME     TAX    HEIGHT    WEIGHT

     1     100     33    100000    5000      61        180
     2     200     55     90000    4000      71        150
     3     300     66     81000    3500      78        210

    /*
     _
    / |    ___  __ _ ___
    | |   / __|/ _` / __|
    | |_  \__ \ (_| \__ \
    |_(_) |___/\__,_|___/

    */
    data want;
      infile "d:/txt/twoRow.txt";
      input
       #1 id age income tax
       #2 id height weight
       ;
    run;quit;

    /*___
    |___ \    __      ___ __  ___
      __) |   \ \ /\ / / `_ \/ __|
     / __/ _   \ V  V /| |_) \__ \
    |_____(_)   \_/\_/ | .__/|___/
                       |_|
    */

    %utl_submit_wps64('
    data want;
      infile "d:/txt/twoRow.txt";
      input
       #1 id age income tax
       #2 id height weight
       ;
    run;quit;
    proc print data=want;
    run;quit;
    ');

    The WPS System

    Obs    ID     AGE    INCOME    TAX     HEIGHT    WEIGHT

     1     100     33    100000    5000      61        180
     2     200     55     90000    4000      71        150
     3     300     66     81000    3500      78        210

    /*____
    |___ /    _ __
      |_ \   | `__|
     ___) |  | |
    |____(_) |_|

    */
    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_r64("
    library(stringr);
    library(tidyverse);
    library(sqldf);
    library(SASxport);
    data <- readLines('d:/txt/twoRow.txt');
    two <- NA;
    for ( idx in seq(1, length(data), 2 ) )
       { two[(idx+1)/2] <- gsub(' +',' ',paste(data[idx],data[idx+1], sep=' '))
       };
    chop<-as.data.frame(t(as.data.frame(strsplit(two, ' '))));
    want<-sqldf('
      select
         cast(v1 as integer) as ID
        ,cast(v2 as integer) as AGE
        ,cast(v3 as integer) as INCOME
        ,cast(v4 as integer) as TAX
        ,cast(v5 as integer) as HEIGHT
        ,cast(v6 as integer) as WEIGHT
      from
        chop
      ');
    want;
    write.xport(want,file='d:/xpt/want.xpt');
    ");

    libname xpt xport "d:/xpt/want.xpt";

    proc print data=xpt.want;
    run;quit;

       ID AGE INCOME  TAX HEIGHT WEIGHT
    1 100  33 100000 5000    100     61
    2 200  55  90000 4000    200     71
    3 300  66  81000 3500    300     78

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
