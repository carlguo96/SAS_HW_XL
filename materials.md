HW1

/*1(a)*/
libname CARS515 'C:\Users\Owner\Desktop\SAS';
/*1(b)*/

proc import datafile='C:\Users\Owner\Desktop\R&SAS\SAS\cars.csv'
	out=CARS515.cars dbms=csv replace;
	getnames=yes;
	guessingrows=500;
run;
proc print data=CARS515.cars;


/*1(c)*/

proc import datafile='C:\Users\Owner\Desktop\R&SAS\SAS\new_cars.xls'
	out=CARS515.new_cars dbms=xls replace;
	sheet='cars';
	getnames=yes;
run;

/*1(d) Error: The data doesn't exist..Q:when I use "access" to do that, I can's read .mdb file into SAS,
so I then save the .mdb file as .cvs file,then improt data */

proc import out=CARS515.CARS_DET datatable="Cars_det" dbms=access replace;
	database='C:\Users\Owner\Desktop\R&SAS\SAS\Cars_Det.mdb';
run;




/*1(d) save the .mdb file as .cvs file to improt data */

proc import datafile='C:\Users\Owner\Desktop\R&SAS\SAS\Cars_Det.csv'
	out=CARS515.CARS_DET dbms=csv replace;
	getnames=yes;
run;


/*2(a) From Log: The data set CARS515.CARS_BASIC_INFO has 429 observations and 10 variables.*/
data CARS515.CARS_BASIC_INFO;
	set CARS515.cars CARS515.new_cars;
run;




/*2(b). Firstly, we need to sort the two datasets. Then we can use (in=) within merge statement and
then use the if statement to combine all common variables in each dataset*/
proc sort data=CARS515.CARS_BASIC_INFO; 
	by Model; 
run;
proc sort data=CARS515.CARS_DET; 
	by Model;
run;
data CARS515.ALL_CARS;
	merge CARS515.CARS_BASIC_INFO (in=a1) CARS515.CARS_DET(in=a2);
	by Model;
	if a1 and a2;
run;



/*3(a)*/
proc sort data=CARS515.ALL_CARS;
	by Type MSRP;
run;



/*3(b)use by.type to output the minimum MSRP,also since first.type=1 and last.type=0, 
then we just need the least expensive car, so just output first.type=1 */
data CARS515.ECONOMY_CARS;
 	set CARS515.ALL_CARS;
	by Type MSRP;
	if first.Type=1 then output;
 run;


/*3(c)sort the dataset first and then to do the following*/
proc sort data=CARS515.ALL_CARS; 
	by Type Model;
run;

data CARS515.MODEL_COUNTS(keep=Type totalcounts);
	set CARS515.ALL_CARS;
	by Type;

	retain totalcounts;
	if first.Type then do;
		totalcounts=0;
end;

	totalcounts=totalcounts+1;

if last.Type then output;
run;


/*3(d)*/
proc sort data=CARS515.ALL_CARS;
	by Type MPG_City;
run;



/*3(e).we use the first.Type=1 to find the minimun MPG_City  */
data CARS515.WASTEFUL_CARS;
 	set CARS515.ALL_CARS;
	by Type MPG_City;
	if first.Type=1 then output;
 run;



/*4:Use Proc Export statement to export a dataset to an spss file*/
proc export data=tabdata outfile='C:\Users\Owner\Desktop\SAS\WASTEFUL_CARS'
	dbms=spss replace;
run;



HW2
/*1(a)from data TWO to create a new data set named TWO_NUMERIC. I set the character ID as 11. format and state as 3.format to make sure
SAS read these correct data*/
data TWO;
	input ID $ 11. State $ 3.; 	
datalines; 
334-99-5246 TX
480-86-3211 MD
449-55-9407 VA
345-12-1778 GA
907-63-6280 NY
790-09-9813 WY
319-86-1601 FL
; 
run;
proc print data=TWO;


data TWO_NUMERIC;
	set TWO;
run;

proc print data=TWO_NUMERIC;



/*1(b)using compress function to set values of ID to ssncomp*/
data TWO_NUMERIC;
	set TWO;
	ssncomp=compress(ID,'-');	
run;
proc print data=TWO_NUMERIC;

/*1(c)for this question, if we just write ID=input(ssncomp,11.),we check the contents and find the ID did not change from character
to numeric, thus I need to named a new "ID_OLD", is a chacter variable, aand then in the next step drop it and let the original ID=ID_OLD */
data TWO_D(drop=ID);
	set TWO;
	ssncomp=compress(ID,'-');
	ID_OLD=input(ssncomp,11.);
run;
data TWO_NUMERIC(drop=ID_OLD);
	set TWO_D;
	ID=ID_old;
run;

proc print data=TWO_NUMERIC;
	title 'Character Converted to Numeric';
run;
proc contents data=TWO_NUMERIC;
run;


/*1(d)Since we want to merge dataset ONE and dataset TWO_NUMERIC, then we need to sort these two datasets first, and then merge them together.
By the way, they all have "ID:, thus we need to merge them by ID, then we get the following results*/
data ONE;
input ID LastName $ FirstInit $ 1.; 
datalines; 
509182793 Smith C
319861601 Williams J
345121778 Connor F
480863211 King L
907636280 Franklin D
729082859 Monroe T
835688938 Hall K
;
run;

proc sort data=ONE;
	by ID;
run;


proc sort data=TWO_NUMERIC;
	by ID;
run;

data MERGED_SSNS;
	merge ONE (in=a1) TWO_NUMERIC(in=a2);
	by ID;
	if a1 and a2;
run;

proc print data=MERGED_SSNS;
run;


/*2(a) store data in the excel file and I will read the dataset from .xlsx*/
proc import datafile='C:\Users\Owner\Desktop\SAS\riverplace.xlsx'
	out=riverplace  dbms=xlsx replace;
	sheet='Sheet1';
	getnames=yes;

run;
proc print data=riverplace;
run;

/*2(b)firstly, I nned to sort the dataset by building and then to do the proc means.
From the results, we can find that "South Building has the lowest mean asking price, which is 210.68 */
proc sort data=riverplace;
	by Building;
run;
proc print data=riverplace;


proc means data=riverplace mean;
 var Price;
 class Building;
run;
 
/*2(c)create "eastonly"dataset to show East Building Only */
data eastonly;
	set riverplace(keep=Price Building);
	if Building ne 'East' then delete;

run;

proc print data=eastonly;
	title 'East Building Only';
run;

/*2(d)use IF-THEN to create a numeric variable named "EASTWEST"*/
data riverplace2;
	set riverplace;

	if Building='East' or Building='West' then EASTWEST=1;
	else EASTWEST=0;
run;

proc print data=riverplace2;
	title 'EASTWEST Variable Added';
run;

/*2(e)use DO-END loop to create four new numeric variables 'NORTH','SOUTH','EAST','WEST'*/

/*do-end*/

data riverplace3;
	set riverplace;

	do _n_= 1 to 30;
		if Building='North' then do;
		NORTH=1;SOUTH=0; EAST=0; WEST=0;
	end;
		else if Building='South' then do;
		NORTH=0;SOUTH=1;EAST=0;WEST=0;
	end;
		else if Building='East' then do;
		NORTH=0;SOUTH=0;EAST=1;WEST=0;
	end;
		else if Building='West' then do;
		NORTH=0;SOUTH=0;EAST=0;WEST=1;
	end;
end;
	drop Building;
run;

proc print data=riverplace3;
	title 'Indicator Variables Added';
run;







HW3
**HW3 SAS Lin_Xu';


/*1(a) create a permanent SAS lib called "diet"*/
libname diet 'C:\Users\Owner\Desktop\SAS';

/*import dataset into "diet"lib*/
data diet.one;
input Lipid_level Age Fat_content_category Gender;
cards;
   0.73      1      1	1
   0.67      1      2	1
   0.15      1      3	1
   0.86      2      1	1
   0.67      2      2	1
   0.15      2      3	1
   0.94      3      1	1
   0.81      3      2	1
   0.26      3      3	1
   0.23	 4	  1	2
   1.40      4      1	1
   1.32      4      2	1
   0.15      4      3	1
   1.62      5      1	1
   1.41      5      2	1
   0.78      5      3	1
   9.78      5      1	1
   1.23	 5	  1	2
;
run;

proc print data=diet.one;



/*1(b)create labels and formats for both Age and Fat_content_category variables*/
proc format library=diet;
	value agef
		1='15-24'
		2='25-34'
		3='35-44'
		4='45-54'
		5='55-64';
	value Fat_content_categoryf
		1='extremely low'
		2='fairly low'
		3='moderately low';
		
run;
**reading in permanent format library;
options fmtsearch=(diet);
**read the new label variable name in the original dataset;
data diet.newone;
	set diet.one;
	label age='age' Fat_content_category ='Fat_content_category'; 
	format age agef. Fat_content_category Fat_content_categoryf.;
run;


/*1(c)plot a boxplot using ods for var "lipid_level"*/;

ods rtf file = "SAS_HW3_LinXu_word.rtf" style=styles.printer;
ods graphics on;
proc sgplot data=diet.newone;
	vbox lipid_level/ fillattrs=(color=blue);
	title 'boxplot of Lipid_level';
run;title;

/*Comment: we can find that there is one outlier of lipid_level. which is significantly higher than any other data point*/




/*1(d)plot a histogram using ods for age and gender=male*/;

Proc sgplot Data=diet.newone(where=(gender=1));
 	HISTOGRAM age / fillattrs=(color=red);
 	format age agef.;
 	title "Histogram of age";
run; title;





 /*1(e)dot plot(sgplot)using ods for the mean lipid_level by age for males only*/

proc sgplot data=diet.newone(where=(gender=1));;
 	dot age/ response=Lipid_level stat=mean ; 
  	title "dotplot for the mean lipid_level by age group for males";
run;title;




/*1(f)create a new dataset and remove Female and remove any outlier for lipid_level*/
**we can use sql to do that;
proc sql;
	create table diet.max as select *,max(Lipid_level) as maxlip from diet.newone;
quit;

data diet.two(where=(gender=1));
	set diet.max;
	if Lipid_level<maxlip then output;
	drop maxlip;
run;

proc print data=diet.two;
run;




/*1(g)plot using ods of lipid level by fat_content_category for each age group*/

 proc sgplot data=diet.two;                
 	series x=Fat_content_category  y=Lipid_level / group=age; 
 	scatter x=Fat_content_category  y=Lipid_level / group=age markerattrs=(symbol=star);
 	title "Lipid_Level by Fat_Content_Category for each Age Group";
run; title;




/*1(h)summary table using ods of lipid_level for each age group*/

**we need to sort the dataset by age first;
proc sort data=diet.newone;
	by age;
run;
proc means data=diet.newone mean median N std;
	by age;                                                
	var Lipid_level;   
	title "summary lipid_level for each age group"; 
run;title;                           





/*1(i)summary table using ods of lipid_level for fat_content_category */

**we need to sort the dataset by fat_content_category first;
proc sort data=diet.newone;
	by Fat_content_category;
run;
proc means data=diet.newone N mean std;
	by Fat_content_category;                                                 
	var Lipid_level;      
	title "summary lipid_level for each Fat_content_category"; 
run;title;                           






************************************************************************************;
/*2(a) create a permanent SAS lib called "flu"*/
libname flu 'C:\Users\lx44\sas';
/*import dataset into "flu"lib*/
data flu.data1;
input Flu_Shot_Status Age Health_Awareness_Index Gender;
cards;
  0    59    52    0
  0    61    55    1
  1    82    51    0
  0    51    70    0
  0    53    70    0
  0    62    49    1
  0    51    69    1
  0    70    54    1
  0    71    65    1
  0    55    58    1
  0    58    48    0
  0    53    58    1
  0    72    65    0
  0    56    68    0
  0    56    83    0
  0    81    68    0
  0    62    44    0
  0    49    70    0
  0    56    69    1
  0    50    74    0
  0    53    57    0
  0    56    64    1
  0    56    67    1
  0    50    83    1
  0    52    48    1
  0    52    81    0
  0    67    53    1
  0    51    61    0
  0    70    51    0
  0    64    51    0
  0    61    65    1
  0    53    51    0
  0    77    54    1
  0    73    64    1
  0    67    69    0
  0    50    71    0
  1    80    38    0
  1    75    51    0
  0    65    54    1
  0    60    59    1
  1    68    57    1
  0    61    63    0
  1    62    48    0
  0    53    58    0
  0    72    56    0
  0    54    59    0
  1    59    75    0
  0    61    48    0
  0    50    79    1
  0    48    66    0
  0    52    57    1
  0    54    68    0
  0    62    48    0
  0    71    60    0
  1    65    63    0
  0    49    61    0
  0    58    57    0
  0    62    69    0
  0    69    38    1
  1    56    50    1
  1    76    45    1
  0    51    72    0
  0    64    51    0
  0    57    62    1
  0    51    81    0
  0    81    55    1
  0    50    77    0
  0    64    65    1
  0    64    53    1
  1    59    49    1
  0    53    65    0
  0    63    58    0
  0    59    60    1
  1    70    57    1
  0    72    37    0
  0    68    49    0
  0    75    55    1
  0    57    60    0
  0    67    57    1
  0    59    56    1
  0    55    58    0
  1    75    64    1
  0    66    51    0
  0    67    59    0
  0    59    61    0
  0    78    49    1
  0    59    49    0
  0    68    55    0
  1    59    61    1
  0    68    50    1
  1    78    47    1
  0    55    73    1
  1    71    45    1
  0    51    45    0
  0    65    59    0
  0    54    61    1
  0    79    52    0
  0    64    50    0
  1    82    46    1
  0    64    67    0
  0    70    56    1
  1    59    50    0
  0    59    56    1
  0    63    61    1
  0    48    74    0
  0    61    78    0
  0    51    68    0
  0    48    71    0
  1    71    58    1
  0    51    57    0
  0    57    51    1
  0    49    74    0
  0    67    56    1
  0    73    57    0
  0    73    65    0
  0    56    47    0
  0    48    69    1
  0    50    71    0
  0    50    76    1
  0    66    60    1
  0    53    75    1
  0    50    65    1
  1    51    42    0
  0    68    66    1
  1    72    49    1
  0    51    58    1
  0    62    61    1
  0    60    55    0
  0    67    60    1
  0    70    54    1
  0    55    63    1
  0    66    56    0
  0    65    59    1
  0    84    52    1
  0    58    63    0
  1    68    57    1
  0    51    59    1
  0    67    53    1
  0    52    67    0
  0    68    62    0
  0    76    63    1
  0    54    62    1
  0    50    52    1
  0    63    58    0
  0    77    49    1
  0    60    65    1
  0    51    55    0
  0    51    60    1
  0    66    51    1
  0    52    67    0
  0    66    64    1
  0    56    55    1
  0    49    58    0
  0    67    66    0
  0    57    64    1
  0    56    66    0
  1    76    22    1
  1    68    32    0
  1    73    56    1
;





/*2(b)create labels and formats for both flu shot status and gender variables*/
proc format library=flu;
	value flushotf
		1='received a flu shot'
		0='did not receive a flu shot';
	value genderf
		1='male'
		0='female';
run;
**reading in permanent format library;
options fmtsearch=(flu);
data flu.newdata1;
	set flu.data1;
	label Flu_Shot_Status='Flu_Shot_Status' Gender='Gender'; 
	format Flu_Shot_Status flushotf. gender genderf. ;
run;






/*2(c)do separate histograms depending flu shot status*/

Proc sgplot Data=flu.newdata1(where=(age>=50 && age<=70 && gender=1 && Flu_Shot_Status=1));
 	title "Histogram for H_A_I when males received a flu shot";
HISTOGRAM Health_Awareness_Index / fillattrs=(color=yellow);
density  Health_Awareness_Index;
Run;title;
Proc sgplot Data=flu.newdata1(where=(age>=50 && age<=70 && gender=1 && Flu_Shot_Status=0));
 	title "Histogram for H_A_I when males did not receive a flu shot";
HISTOGRAM Health_Awareness_Index / fillattrs=(color=pink);
density  Health_Awareness_Index;
Run;title;



/*2(d)QQ plot for Health_Awareness_Index */

Proc Univariate Data=flu.newdata1 noprint;
 	Var Health_Awareness_Index;
 	title "Qq Plot for Health_Awareness_Index";
	qqplot Health_Awareness_Index/normal(mu=est sigma=est)  ;
run; title;







/*2(e)scatter plot for Health_Awareness_Index and Age for males only*/

proc sgplot data=flu.newdata1(where=(gender=1));
	scatter x=Health_Awareness_Index y=age;
	title "Scatter plot of Health_Awareness_Index and Age for males only";
run;title;






/*2(f)using sgpanel*/
proc sgpanel data=flu.newdata1;
	panelby gender Flu_Shot_Status / layout=lattice;
	scatter x=Health_Awareness_Index y=age;
	format Flu_Shot_Status flushotf. gender genderf.; 
	title "Health_Awareness_Index & Age Plot";
run;




/*2(g)make a new variable Agegroup*/ 
**firstly, we need to use proc means to find the Q1 median and Q3 for age group;
proc means data=flu.newdata1 Q1 median Q3;
	var age;
	title "age statistics";
run;


**create the new var Agegroup;
data flu.data2;
	set flu.newdata1;
	if age<=53 then agegroup=1;
	else if age<=61 then agegroup=2;
	else if age<=68 then agegroup=3;
	else if age>68 then agegroup=4;
run;

proc sort data=flu.data2;
	by Agegroup; 
run;

proc means data=flu.data2 N mean median std;
	by Agegroup ;
	var Health_Awareness_Index;
	title justify=left "Summary of Health_Awareness_Index for Each AgeGroup";
run;

ods graphics off;
ods rtf close;
