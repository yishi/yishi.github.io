---
title: sas statistical model summary exercise
author: ''
date: '2015-05-22'
slug: sas-statistical-model-summary-exercise
categories:
  - SAS
tags:
  - exercise
---

```{sas}
**********************************************************************;
*** from the exercises of <<common statistical methods for clinical***;
*** research with sas examples>>                                  ***;
**********************************************************************;

data trial;
input trt $ center pat sex $ age score @@;
   resp = (score gt 0);
    
    if (     score =   0) then sev = 0; 
    if ( 1 le score le 30) then sev = 1; 
    if (31 le score le 69) then sev = 2; 
    if (     score ge 70) then sev = 3; 
    datalines;
A 1 101 M 55  5  A 1 104 F 27 0   A 1 106 M 31 35
A 1 107 F 44 21   A 1 109 M 47 15   A 1 111 F 69 70
A 1 112 F 31 10   A 1 114 F 50  0   A 1 116 M 32 20
A 1 118 F 39 25   A 1 119 F 54  0   A 1 121 M 70 38
A 1 123 F 57 55   A 1 124 M 37 18   A 1 126 F 41  0
A 1 128 F 48  8  A 1 131 F 35 0   A 1 134 F 28  0
A 1 135 M 27 40   A 1 138 F 42 12   A 2 202 M 58 68
A 2 203 M 42 22   A 2 206 M 26 30   A 2 207 F 36  0
A 2 210 F 35 25   A 2 211 M 51  0   A 2 214 M 51 60
A 2 216 F 42 15   A 2 217 F 50 50   A 2 219 F 41 35
A 2 222 F 59  0  A 2 223 F 38 10   A 2 225 F 32  0
A 2 226 F 28 16   A 2 229 M 42 48   A 2 231 F 51 45
A 2 234 F 26 90   A 2 235 M 42  0   A 3 301 M 38 28
A 3 302 M 41 20   A 3 304 M 65 75   A 3 306 F 64  0
A 3 307 F 30 30   A 3 309 F 64  5   A 3 311 M 39 80
A 3 314 F 57 85   A 3 315 M 61 12   A 3 318 F 45 95
A 3 319 F 34 26   A 3 321 M 39 10   A 3 324 M 27  0
A 3 325 F 56 35   B 1 102 M 19 68   B 1 103 F 51 10
B 1 105 M 45 20   B 1 108 F 44 65   B 1 110 M 32 25
B 1 113 M 61 75   B 1 115 M 45 83   B 1 117 F 21  0
B 1 120 F 19 55   B 1 122 F 38  0   B 1 125 M 37 72
B 1 127 F 53 40   B 1 129 M 48  0   B 1 130 F 36 80
B 1 132 M 49 20   B 1 133 F 28  0   B 1 136 F 34 45
B 1 137 F 57 95   B 1 139 F 47 40   B 1 140 M 29  0
B 2 201 F 63 10   B 2 204 M 36 49   B 2 205 M 36 16
B 2 208 F 48 12   B 2 209 F 42 40   B 2 212 F 32  0
B 2 213 M 24 88   B 2 215 M 40 59   B 2 218 M 31 24
B 2 220 F 45 72   B 2 221 F 27 55   B 2 224 M 56 70
B 2 227 F 41  0  B 2 228 F 24 65   B 2 230 M 44 30
B 2 232 M 37 32   B 2 233 F 33  0   B 3 303 M 40 26
B 3 305 M 46 15   B 3 308 M 59 82   B 3 310 F 62 38
B 3 312 M 52 40   B 3 313 F 33 40   B 3 316 M 62 87
B 3 317 M 52 60   B 3 320 F 32  2   B 3 322 F 43  0
B 3 323 F 51 35
;
proc sort data = trial;
    by pat;
run;
proc print data = trial;
    var pat trt center sex age resp sev score;
    title 'Printout of Data Set TRIAL, Sorted by PAT';
run;


*==============================================================*
|  statistical analyses for continuous numeric variable      |
|                     score                                  |
*==============================================================*;

  
proc sort data = trial;
    by trt;
run;
proc univariate normal data = trial;
   by trt;
   var score;
   run;
proc ttest data = trial;
  class trt;
  var score;
run;

proc nparlway wilcoxon data = trial;
  class trt;
  var score;
  run;

  
proc glm data = trial;
  class trt;
  model score = trt;
  means trt / hovtest t ;
  run;

    
proc glm data = trial;
  class trt center;
  model score = trt center trt*center / ss3;
        lsmeans trt / stderr pdiff ;
      
run;
quit;

    
proc glm data = trial;
  class trt center;
  model score = trt center / ss3;
run;
quit;

proc glm data = trial;
  class trt center sex;
 * model score = trt center sex trt*center trt*sex center*sex trt*center*sex / ss3;
  model score = trt | center | sex;
       lsmeans trt / stderr pdiff ;
run;
quit;

proc glm data = trial;
  class trt center sex;
  model score = trt center sex / ss3;
run;
quit;

ods graphics on ;
proc glm data = trial;
  model score = age / solution;
run;
quit;
ods graphics off ;

proc glm data = trial;
  class trt;
  model score = trt age trt*age;
  
run;
quit;

*** ods graphics on;
*** proc glm data = trial plots = ancovaplot;
ods graphics on ;
proc glm data = trial;
  class trt;
  model score = trt age / solution;
        lsmeans trt /pdiff stderr ;
run;
quit;
ods graphics off ;
*** ods graphics off;

proc glm data = trial;
  class trt center;
  model score = trt age center / solution;
        lsmeans trt /diff;
run;
quit;


*==============================================================*
|  statistical analyses for dichotomous variable             |
|                     resp                                   |
*==============================================================*;
*********************************************;

proc freq data = trial;
  tables trt*resp / chisq nopercent nocol ;
run;

proc freq data = trial;
  tables trt * resp / fisher nopercent nocol ;
run;

proc freq data = trial;
  tables center *trt* resp / cmh nopercent nocol ;
run;

proc logistic data = trial;
  class trt / param = ref;
  model resp = trt;
run;

proc logistic data = trial;
  class trt / param = ref;
  model resp = trt age;
run;

proc logistic data = trial;
  class trt sex center / param = ref;
  model resp = trt age sex center trt*center;
run;


*==============================================================*
|  statistical analyses for ordinal categorical variable     |
|                     sev                                    |
*==============================================================*;
**********************************************;
*** Using the SEV response variable;

proc freq data = trial;
  tables trt * sev / chisq nopercent nocol ;
run;

proc freq data = trial;
  tables trt * sev / exact nopercent nocol ;
run;

proc freq data = trial;
  tables trt * sev / cmh nopercent nocol ;
run;

proc freq data = trial;
  tables trt * sev / cmh nopercent nocol scores = modridit;
run;

proc nparlway wilcoxon data = trial;
  class trt;
  var sev;
run;

proc freq data = trial;
  tables center * trt * sev / cmh nocol nopct;
run;

proc freq data = trial;
  tables center * trt * sev / cmh scores = modridit nocol nopct;
run;

proc logistic data = trial;
  class trt / param = ref;
  model sev = trt;
run;

proc logistic data = trial;
  class trt / param = ref;
  model sev = trt age;
run;

proc logistic data = trial;
  class trt sex center;
  model sev = trt age sex center;
run;

*** 434 449/554 page information will help understand ;
```
Welcome your advises and suggestion!

备注：转移自新浪博客，截至2021年11月，原阅读数66，评论0个。 

