* Encoding: UTF-8.

*******************************************************************************
***************************Research Question 1****************************
********************************************************************************

DATASET ACTIVATE DataSet3.
FREQUENCIES VARIABLES=age
  /STATISTICS=STDDEV MEAN
  /HISTOGRAM
  /ORDER=ANALYSIS.

RECODE age (1 thru 50=1) (ELSE=0) INTO Age_new.
EXECUTE.

USE ALL.
FILTER BY Age_new.
EXECUTE.

FREQUENCIES VARIABLES=sex
  /STATISTICS=STDDEV MEAN
  /HISTOGRAM
  /ORDER=ANALYSIS.

RECODE sex ('male'=0) ('female'=1) (ELSE=SYSMIS) INTO gender.
EXECUTE.

FREQUENCIES VARIABLES=STAI_trait
  /STATISTICS=STDDEV MEAN
  /HISTOGRAM
  /ORDER=ANALYSIS.

FREQUENCIES VARIABLES=pain_cat
  /STATISTICS=STDDEV MEAN
  /HISTOGRAM
  /ORDER=ANALYSIS.

FREQUENCIES VARIABLES=mindfulness
  /STATISTICS=STDDEV MEAN
  /HISTOGRAM
  /ORDER=ANALYSIS.

FREQUENCIES VARIABLES=cortisol_serum cortisol_saliva
  /STATISTICS=STDDEV MEAN
  /HISTOGRAM NORMAL
  /ORDER=ANALYSIS.

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS R ANOVA
  /CRITERIA=PIN(.05) POUT(.10) CIN(95)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=ENTER age STAI_trait pain_cat cortisol_serum cortisol_saliva mindfulness gender
  /SAVE PRED COOK MCIN RESID.

* Chart Builder.
GGRAPH
  /GRAPHDATASET NAME="graphdataset" VARIABLES=ID COO_2 MISSING=LISTWISE REPORTMISSING=NO
  /GRAPHSPEC SOURCE=INLINE
  /FITLINE TOTAL=NO.
BEGIN GPL
  SOURCE: s=userSource(id("graphdataset"))
  DATA: ID=col(source(s), name("ID"), unit.category())
  DATA: COO_2=col(source(s), name("COO_2"))
  GUIDE: axis(dim(1), label("ID"))
  GUIDE: axis(dim(2), label("Cook's Distance"))
  GUIDE: text.title(label("Simple Scatter of Cook's Distance by ID"))
  SCALE: linear(dim(2), include(0))
  ELEMENT: point(position(ID*COO_2))
END GPL.

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) BCOV R ANOVA COLLIN TOL
  /CRITERIA=PIN(.05) POUT(.10) CIN(95)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=ENTER age gender
  /METHOD=ENTER STAI_trait pain_cat cortisol_serum cortisol_saliva mindfulness
  /SAVE PRED COOK MCIN RESID.


REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) R ANOVA COLLIN TOL CHANGE
  /CRITERIA=PIN(.05) POUT(.10) CIN(95)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=ENTER age gender
  /METHOD=ENTER STAI_trait pain_cat cortisol_serum mindfulness
  /SCATTERPLOT=(*ZRESID ,*ZPRED)
  /RESIDUALS HISTOGRAM(ZRESID) NORMPROB(ZRESID)
  /SAVE PRED COOK MCIN RESID.

EXAMINE VARIABLES=RES_4
  /PLOT BOXPLOT STEMLEAF HISTOGRAM NPPLOT
  /COMPARE GROUPS
  /STATISTICS DESCRIPTIVES
  /CINTERVAL 95
  /MISSING LISTWISE
  /NOTOTAL.

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) R ANOVA COLLIN TOL CHANGE
  /CRITERIA=PIN(.05) POUT(.10) CIN(95)
  /NOORIGIN 
  /DEPENDENT PRE_4
  /METHOD=ENTER age gender
  /METHOD=ENTER RES_4
  /SCATTERPLOT=(*ZRESID ,*ZPRED)
  /RESIDUALS HISTOGRAM(ZRESID) NORMPROB(ZRESID)
  /SAVE PRED COOK MCIN RESID.
 
 ************************************************************************************
 *****************************Research question 2*******************************
 ************************************************************************************

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) R ANOVA CHANGE
  /CRITERIA=PIN(.05) POUT(.10)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=BACKWARD age gender weight mindfulness cortisol_serum STAI_trait pain_cat.

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) R ANOVA CHANGE SELECTION
  /CRITERIA=PIN(.05) POUT(.10) CIN(95)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=ENTER age gender mindfulness cortisol_serum pain_cat
  /SCATTERPLOT=(*ZRESID ,*ZPRED)
  /RESIDUALS HISTOGRAM(ZRESID) NORMPROB(ZRESID)
  /SAVE PRED ZPRED COOK MCIN RESID ZRESID.

*******Changing dataset to home assignment data 2*****************

RECODE sex ('female'=1) ('male'=0) (ELSE=SYSMIS) INTO gender.
EXECUTE.

COMPUTE pred_pain_theory_model=4.761 + (-0.079 * age) + (-0.326 *gender) + (0.017*STAI_trait) + 
    (0.057*pain_cat) + (0.392*cortisol_serum) + (0.282*mindfulness).
EXECUTE.

COMPUTE pain_pred_backwards=4.915 + (-0.075 * age) + (-0.309 *gender) + (0.066*pain_cat) + 
    (0.411*cortisol_serum) + (0.290*mindfulness).
EXECUTE.

COMPUTE residual_theory_model=pain - pred_pain_theory_model.
EXECUTE.

COMPUTE residual_backwards_model=pain - pain_pred_backwards.
EXECUTE.

COMPUTE theory_res_sq=residual_theory_model*residual_theory_model.
EXECUTE.

COMPUTE backwards_res_sq=residual_backwards_model * residual_backwards_model.
EXECUTE.

COMPUTE pain_sq=pain * pain.
EXECUTE.

DESCRIPTIVES VARIABLES=theory_res_sq backwards_res_sq pain_sq
  /STATISTICS=MEAN SUM STDDEV MIN MAX.

**testing backwards vs. initial**

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) R ANOVA CHANGE SELECTION
  /CRITERIA=PIN(.05) POUT(.10)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=ENTER age gender weight mindfulness cortisol_serum STAI_trait pain_cat
  /METHOD=REMOVE STAI_trait weight 
  
**testing backwards against theoretical**

REGRESSION
  /MISSING LISTWISE
  /STATISTICS COEFF OUTS CI(95) R ANOVA CHANGE SELECTION
  /CRITERIA=PIN(.05) POUT(.10)
  /NOORIGIN 
  /DEPENDENT pain
  /METHOD=ENTER age gender mindfulness cortisol_serum STAI_trait pain_cat
  /METHOD=REMOVE STAI_trait
  
******************************************************************************
***********************RESEARCH QUESTION 3*************************
******************************************************************************

RECODE sex ('female'=1) ('male'=0) (ELSE=SYSMIS) INTO gender.
EXECUTE.

FREQUENCIES VARIABLES=pain cortisol_serum hospital
  /STATISTICS=STDDEV MINIMUM MEAN
  /HISTOGRAM
  /ORDER=ANALYSIS.

GGRAPH
  /GRAPHDATASET NAME="graphdataset" VARIABLES=cortisol_serum pain hospital MISSING=LISTWISE 
    REPORTMISSING=NO
  /GRAPHSPEC SOURCE=INLINE
  /FITLINE TOTAL=NO SUBGROUP=NO.
BEGIN GPL
  SOURCE: s=userSource(id("graphdataset"))
  DATA: cortisol_serum=col(source(s), name("cortisol_serum"))
  DATA: pain=col(source(s), name("pain"))
  DATA: hospital=col(source(s), name("hospital"), unit.category())
  GUIDE: axis(dim(1), label("cortisol_serum"))
  GUIDE: axis(dim(2), label("pain"))
  GUIDE: legend(aesthetic(aesthetic.color.interior), label("hospital"))
  GUIDE: text.title(label("Grouped Scatter of pain by cortisol_serum by hospital"))
  ELEMENT: point(position(cortisol_serum*pain), color.interior(hospital))
END GPL.

MIXED pain BY sex WITH age STAI_trait pain_cat cortisol_serum mindfulness
  /CRITERIA=CIN(95) MXITER(100) MXSTEP(10) SCORING(1) SINGULAR(0.000000000001) HCONVERGE(0, 
    ABSOLUTE) LCONVERGE(0, ABSOLUTE) PCONVERGE(0.000001, ABSOLUTE)
  /FIXED=sex age STAI_trait pain_cat cortisol_serum mindfulness | SSTYPE(3)
  /METHOD=REML
  /PRINT=SOLUTION
  /RANDOM=INTERCEPT | SUBJECT(hospital) COVTYPE(VC)
  /SAVE=FIXPRED RESID.

DESCRIPTIVES VARIABLES=FXPRED_1
  /STATISTICS=VARIANCE.

****DATASET 4****

RECODE sex ('female'=1) ('male'=0) (ELSE=SYSMIS) INTO gender.
EXECUTE.

COMPUTE pred_pain2=2.33 + (-0.026 * age) + (-0.382 * gender) + (-0.014 * STAI_trait) + (0.082 * 
    pain_cat) + (0.533 * cortisol_serum) + (0.239 * mindfulness).
EXECUTE.

COMPUTE residual_pain2=pain - pred_pain2
EXECUTE.

COMPUTE TSS2=(pain-4.99210399)*(pain-4.99210399).
EXECUTE.

COMPUTE RSS3 =(pain-pred_pain2)*(pain-pred_pain2).
EXECUTE.

DESCRIPTIVES VARIABLES=TSS2 RSS3
  /STATISTICS=MEAN SUM STDDEV MIN MAX.
