R Script V5
================
Alex Popescu
2023-05-12

V5 - Major overhaul and data/analysis improvement.

# MSC Thesis Script

> I used the advanced filter tool in BORIS to extract all instances
> where a behavior occurred in the presence and absence of a sentinel.
> This generated 6 tables. I then manually inputted the ID, generalized
> environment, number of **recorded** crows and whether or not the site
> was baited in the recording. I additionally removed observations in
> agricultural settings, and observations past September 29th.

### Reasoning

> The number of crows in the area is not a good estimate of group size.
> The number of foraging members during the foraging event should be
> better. Whether or not the site was baited will likely affect the
> duration of movement, since crows will need to look for food in
> non-baited recordings, while the crows have easy access to a large
> concentration of food in baited recordings. Therefore, the baited
> crows are expected to move less than non-baited crows. I removed the
> observations in agricultural settings because they had no sentinels
> present. This could be talked about in the discussion: agricultural
> fields may lack perches for individuals to act as sentinels. Vineyards
> may have some structures, but the visibility is limited when the vines
> are laden with fruit.  
> I removed observations past Sept 29 because I was told behaviors can
> change wildly in the winter. The number of crows spotted decreased
> past that date, suggesting the migrants have left or were leaving. To
> not have to include time of year (I already have part of the breeding
> season), I decided to simplify.  

### Data Management

> After combining the 6 tables into a long one, labeling the behaviors
> and the presence of a sentinel on the way, I made sure that the
> headers were factors.  

    ## 'data.frame':    3486 obs. of  14 variables:
    ##  $ Observation.id : Factor w/ 25 levels "020 - FVM - 2 crows - No Bait",..: 2 2 2 2 2 2 2 4 4 4 ...
    ##  $ Duration       : num  2.029 3.995 7.994 0.498 7.773 ...
    ##  $ ID             : Factor w/ 22 levels "20","24","25",..: 2 2 2 2 2 2 2 4 4 4 ...
    ##  $ CODED_G.ENV    : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ G.Env          : Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ CODED_NB_CROWS : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ Number.of.Crows: num  2 2 2 2 2 2 2 1 1 1 ...
    ##  $ CODED_BAIT     : num  0 0 0 0 0 0 0 1 1 1 ...
    ##  $ Baited         : Factor w/ 2 levels "N","Y": 1 1 1 1 1 1 1 2 2 2 ...
    ##  $ CODED_SENTINEL : num  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ Sentinel       : Factor w/ 2 levels "N","Y": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Behavior       : Factor w/ 3 levels "HD","HU","M": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ CODED_BEHAVIOR : num  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ LDur           : num  1.108 1.608 2.197 0.404 2.172 ...

> Good news! Observation ID, bait, presence of a sentinel and behavior
> are indeed recognized as factors! Number of crows and bout duration as
> a numeric variable.

# Bout Duration

## Figures

### Histogram

> I first decided to look at the distribution of my durations. I set up
> a histogram with the distibution of durations across all behaviors. I
> had to log-transform the duration to get a better distribution
> (log(Duration+1)).  

![](MSC-R-Script_files/figure-gfm/Duration%20Histograms%20Combined-1.png)<!-- -->

![](MSC-R-Script_files/figure-gfm/Crow%20Histogram-1.png)<!-- -->

> I should use a robust linear mixed model. LMMs by themselves are
> robust against violations of their assumptions (I found an article
> about this), but I will nonetheless use ‘robustlmm’ to make my model,
> since their computations for weights are apparently better.

### Boxplot

![](MSC-R-Script_files/figure-gfm/Crow%20boxplot-1.png)<!-- -->

### Violin plot

![](MSC-R-Script_files/figure-gfm/Crow%20Violin%20plot-1.png)<!-- -->

> These plots are nice, but it is hard to really see the underlying
> trends. To best visualise them, I’ll next plot a dot plot.

### The Dot Plot

![](MSC-R-Script_files/figure-gfm/Dot%20Plot%20V2-1.png)<!-- -->

> Very cool! I think the last plot is the most informative. We can see
> that the presence of a sentinel increases the duration of bouts in
> commercial areas. In green areas, however, this effect is not
> observed. In fact, the bouts of Alert behavior seem significantly
> reduced in green areas and in the presence of a sentinel.

## The Tests

<!-- ### PCAs -->
<!-- ```{r Bout Duration PCA packages, include=F} -->
<!-- library("FactoMineR") -->
<!-- library("factoextra") -->
<!-- ``` -->
<!-- ```{r Bout Duration PCA Book keeping, include=F} -->
<!-- PCA.Crow.Data<-Crow[,c(4,6,8,10,12:13)] -->
<!-- head(PCA.Crow.Data) -->
<!-- ``` -->
<!-- ```{r Bout Duration PCA, echo=F} -->
<!-- PCA.Crow<-PCA(PCA.Crow.Data[,-5], ncp=5) -->
<!-- ``` -->
<!-- >I'm new to PCAs, so bear with me please. From what I see, presence of bait and sentinel, and the number of crows all contibute to PC1, while Generalized environment seems to be contributing to PC2. The behavior appears to contribute to PC2, but not a lot. -->
<!-- #### Eigenvalues -->
<!-- >Let's have a look at the eigenvalues. -->
<!-- ``` {r Bout Duration PCA eigenvalues, echo = F} -->
<!-- Crow.eig.val<-get_eigenvalue(PCA.Crow) -->
<!-- Crow.eig.val -->
<!-- ``` -->
<!-- >Pretty small values. The second column shows how much contribution each dimension has to explain the total variance in the data. Let's quickly plot that. -->
<!-- #### Scree Plot -->
<!-- ```{r Bout Duration PCA Scree Plot, echo = F} -->
<!-- fviz_eig(PCA.Crow, addlabels=T, ylim = c(0,40)) -->
<!-- ``` -->
<!-- >Pretty cool. The first 3 PCs explain ~77% of the variance. Adding the 4th PC explains ~93% of the variation. -->
<!-- #### Contribution of variables to Principal Components -->
<!-- ```{r Bout Duration PCA var results, include = F} -->
<!-- Crow.var<-get_pca_var(PCA.Crow) -->
<!-- ``` -->
<!-- ```{r Bout Duration PCA Contribution Dim 1, echo=F} -->
<!-- fviz_contrib(PCA.Crow, choice="var", axes = 1, top = 20) -->
<!-- ``` -->
<!-- >The red dotted line represents the expected average contribution, assuming the contribution of the variables were uniform. In other words, the line is equal to 1/the number of variables; in this case 1/5 or 20%. -->
<!-- > From this plot, I can see that, as I saw previously in the correlation circle, the number of crows, the presence of a sentinel and the presence of bait contribute the most to PC1 (Dimension 1) -->
<!-- ```{r Bout Duration PCA Contribution Dim 2, echo=F} -->
<!-- fviz_contrib(PCA.Crow, choice="var", axes = 2, top = 20) -->
<!-- ``` -->
<!-- > The generalized environment contributes the most to PC2. -->
<!-- ```{r Bout Duration PCA Contribution Dim 3, echo=F} -->
<!-- fviz_contrib(PCA.Crow, choice="var", axes = 3, top = 20) -->
<!-- ``` -->
<!-- >The coded behavior contributes most to PC3. -->
<!-- ```{r Bout Duration PCA Contribution Dim 4, echo=F} -->
<!-- fviz_contrib(PCA.Crow, choice="var", axes = 4, top = 20) -->
<!-- ``` -->
<!-- >OK! Now we're getting to the interesting bits. The presence of bait and the presence of a Sentinel contribute most to PC4. This makes me curious to see if there is an interaction between the two factors. -->
<!-- ```{r Bout Duration PCA Contribution Dim 5, echo=F} -->
<!-- fviz_contrib(PCA.Crow, choice="var", axes = 5, top = 20) -->
<!-- ``` -->
<!-- >>The presence of a sentinel and the size of the group contribute most to PC 5. I believe this would imply that the interaction between the two could be interesting to look at. Considering these two variables are also major contributors to PC1, this makes the inclusion of the interaction in the model a must. -->
<!-- >From this, I believe I should look at the interaction between NB_Crows, Presence of a sentinel, and the presence of bait. -->
<!-- #### Plotting with Behaviors -->
<!-- >I'd like to plot the dimensions using the behaviors of the bouts. -->
<!-- ```{r Bout Duration PCA Plot with Behavior PC1-2, echo=F} -->
<!-- fviz_pca_ind(PCA.Crow -->
<!--              , geom.ind="point" -->
<!--              , col.ind=PCA.Crow.Data$Behavior -->
<!--              , palette = cbPalette -->
<!--              , addEllipses = T -->
<!--              , legend.title = "Behaviors" -->
<!--              ) -->
<!-- ``` -->
<!-- >Messy. The mean points of the behaviors are relatively centered. -->
<!-- ```{r Bout Duration PCA Plot with Behavior PC2-3, echo=F} -->
<!-- fviz_pca_ind(PCA.Crow -->
<!--              , axes = c(2,3) -->
<!--              , geom.ind="point" -->
<!--              , col.ind=PCA.Crow.Data$Behavior -->
<!--              , palette = cbPalette -->
<!--              , addEllipses = T -->
<!--              , legend.title = "Behaviors" -->
<!--              ) -->
<!-- ``` -->
<!-- >Very clear separation by behavior type, with 'Head Up' shifted upward, 'Head Down' shifted downward, and 'Moving' in the middle. -->
<!-- ```{r Bout Duration PCA Plot with Behavior PC1-3, echo=F} -->
<!-- fviz_pca_ind(PCA.Crow -->
<!--              , axes = c(1,3) -->
<!--              , geom.ind="point" -->
<!--              , col.ind=PCA.Crow.Data$Behavior -->
<!--              , palette = cbPalette -->
<!--              , addEllipses = T -->
<!--              , legend.title = "Behaviors" -->
<!--              ) -->
<!-- ``` -->
<!-- >Good separation, but the concentration ellipses have increased in size. Curious. -->
<!-- #### Biplot -->
<!-- >Finally, let's look at the biplot. From what I understand, it is essentially the correlation circle overlaid onto the data. -->
<!-- ```{r Bout Duration PCA Biplot PC1-2, echo=F} -->
<!-- fviz_pca_biplot(PCA.Crow -->
<!--                 ,geom.ind = "point" -->
<!--                 , repel = T -->
<!--                 , col.ind=PCA.Crow.Data$Behavior -->
<!--                 , addEllipses = T -->
<!--                 , label = "var" -->
<!--                 , palette=cbPalette -->
<!--                 , col.var="black" -->
<!--                 , legend.title = "Behavior") -->
<!-- ``` -->
<!-- ```{r Bout Duration PCA Biplot PC2-3, echo=F} -->
<!-- fviz_pca_biplot(PCA.Crow -->
<!--                 , axes = c(2,3) -->
<!--                 , geom.ind = "point" -->
<!--                 , repel = T -->
<!--                 , col.ind=PCA.Crow.Data$Behavior -->
<!--                 , addEllipses = T -->
<!--                 , label = "var" -->
<!--                 , palette=cbPalette -->
<!--                 , col.var="black" -->
<!--                 , legend.title = "Behavior") -->
<!-- ``` -->
<!-- ```{r Bout Duration PCA Biplot PC1-3, echo=F} -->
<!-- fviz_pca_biplot(PCA.Crow -->
<!--                 , axes = c(1,3) -->
<!--                 , geom.ind = "point" -->
<!--                 , repel = T -->
<!--                 , col.ind=PCA.Crow.Data$Behavior -->
<!--                 , addEllipses = T -->
<!--                 , label = "var" -->
<!--                 , palette=cbPalette -->
<!--                 , col.var="black" -->
<!--                 , legend.title = "Behavior") -->
<!-- ``` -->
<!-- >Nice! I think I've done the PCA analysis to the best of my abilities, but I am very much open to comments and feedback! -->
<!-- >From what I can gather, I should model the data as follows:/ -->
<!-- >LDur~Behavior+NB_Crows*Bait*Sentinel+G.Env+(1|ID) -->

### Assumption testing first

> I used robustlmm::rlmer to create the models. It takes longer to run
> than the regular lmer test because of the computations it makes to
> weigh the model. I then plotted the models to test the assumptions of
> the model.

> Behavior is expected to be affected by both the presence of a sentinel
> and the environment the crows are in. I am therefore curious to see
> the interactions between these factors. The presence of bait and
> number of crows are not linked with the presence of a sentinel or the
> environment. Arguably, larger groups are more likely to have sentinels
> than not, but I don’t believe the interaction to be important here.

### RLMM

![](MSC-R-Script_files/figure-gfm/Duration%20RLMM-1.png)<!-- -->![](MSC-R-Script_files/figure-gfm/Duration%20RLMM-2.png)<!-- -->![](MSC-R-Script_files/figure-gfm/Duration%20RLMM-3.png)<!-- -->

> That took pretty long to run. Let’s next see the results of the model.
> I don’t see huge deviations from the assumptions. According to the
> error message, my fixed-effect model matrix is rank deficient. I don’t
> believe I need to worry too much about this warning.

    ## Robust linear mixed model fit by DAStau 
    ## Formula: LDur ~ Behavior + Sentinel * G.Env + Sentinel * CODED_NB_CROWS +      Baited + Sentinel + G.Env + (1 | ID) 
    ##    Data: Crow 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -2.0274 -0.6885 -0.1257  0.6715  6.3157 
    ## 
    ## Random effects:
    ##  Groups   Name        Variance Std.Dev.
    ##  ID       (Intercept) 0.004339 0.06587 
    ##  Residual             0.229408 0.47897 
    ## Number of obs: 3486, groups: ID, 22
    ## 
    ## Fixed effects:
    ##                           Estimate Std. Error t value
    ## (Intercept)                0.96964    0.05049  19.206
    ## BehaviorHU                -0.21559    0.01919 -11.234
    ## BehaviorM                 -0.06476    0.02225  -2.910
    ## SentinelY                  0.10910    0.04895   2.229
    ## G.EnvGreen Area            0.12938    0.04750   2.724
    ## CODED_NB_CROWS1            0.14406    0.09052   1.591
    ## BaitedY                   -0.11227    0.04242  -2.646
    ## SentinelY:G.EnvGreen Area -0.17058    0.05711  -2.987
    ## SentinelY:CODED_NB_CROWS1 -0.06700    0.09020  -0.743
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr) BhvrHU BehvrM SntnlY G.EnGA CODED_ BaitdY SY:G.A
    ## BehaviorHU  -0.211                                                 
    ## BehaviorM   -0.228  0.464                                          
    ## SentinelY   -0.240 -0.012 -0.001                                   
    ## G.EnvGrenAr -0.577  0.010  0.020  0.375                            
    ## CODED_NB_CR -0.114  0.005 -0.011  0.283  0.311                     
    ## BaitedY     -0.629  0.004  0.062 -0.134  0.000 -0.270              
    ## SntnY:G.EGA  0.195  0.010 -0.004 -0.815 -0.604 -0.237  0.162       
    ## SY:CODED_NB  0.131 -0.006 -0.002 -0.529 -0.213 -0.843  0.084  0.426
    ## 
    ## Robustness weights for the residuals: 
    ##  2858 weights are ~= 1. The remaining 628 ones are summarized as
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.213   0.545   0.757   0.722   0.911   0.999 
    ## 
    ## Robustness weights for the random effects: 
    ##  20 weights are ~= 1. The remaining 2 ones are
    ##     9    15 
    ## 0.995 0.838 
    ## 
    ## Rho functions used for fitting:
    ##   Residuals:
    ##     eff: smoothed Huber (k = 1.345, s = 10) 
    ##     sig: smoothed Huber, Proposal 2 (k = 1.345, s = 10) 
    ##   Random Effects, variance component 1 (ID):
    ##     eff: smoothed Huber (k = 1.345, s = 10) 
    ##     vcp: smoothed Huber, Proposal 2 (k = 1.345, s = 10)

> A whole lot of numbers appear and no p-values. I want it to look
> better.

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">
Model Results with Interactions
</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">
 
</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
Effects of on duration of behaviors
</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">
Predictors
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Estimates
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
p
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
(Intercept)
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.97
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.87 – 1.07
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Behavior \[HU\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.22
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.25 – -0.18
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Behavior \[M\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.06
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.11 – -0.02
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.004</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.11
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.01 – 0.21
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.026</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
G Env \[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.13
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.04 – 0.22
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.006</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
CODED NB CROWS \[1\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.14
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.03 – 0.32
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.112
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Baited \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.11
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.20 – -0.03
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.008</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\] × G Env<br>\[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.17
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.28 – -0.06
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.003</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\] × CODED NB<br>CROWS \[1\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.07
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.24 – 0.11
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.458
</td>
</tr>
<tr>
<td colspan="4" style="font-weight:bold; text-align:left; padding-top:.8em;">
Random Effects
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
σ<sup>2</sup>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.23
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
τ<sub>00</sub> <sub>ID</sub>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.00
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
ICC
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.02
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
N <sub>ID</sub>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
22
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">
Observations
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">
3486
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
Marginal R<sup>2</sup> / Conditional R<sup>2</sup>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.053 / 0.071
</td>
</tr>
</table>

> Oh fun! Adding the three-way interaction caused all but the behavior
> to be non-significant. Let’s try removing the interactions.

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">
Model Results without Interactions
</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">
 
</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
Effects of on duration of behaviors
</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">
Predictors
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
Estimates
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
CI
</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">
p
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
(Intercept)
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
1.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.88 – 1.12
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Behavior \[HU\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.22
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.25 – -0.18
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Behavior \[M\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.07
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.11 – -0.02
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.003</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.01
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.05 – 0.07
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.718
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
CODED NB CROWS \[1\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.12
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.01 – 0.24
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.037</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Baited \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.10
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.20 – 0.00
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.059
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
G Env \[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.03
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.06 – 0.13
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.482
</td>
</tr>
<tr>
<td colspan="4" style="font-weight:bold; text-align:left; padding-top:.8em;">
Random Effects
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
σ<sup>2</sup>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.23
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
τ<sub>00</sub> <sub>ID</sub>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.01
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
ICC
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.04
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
N <sub>ID</sub>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
22
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">
Observations
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">
3486
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
Marginal R<sup>2</sup> / Conditional R<sup>2</sup>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.051 / 0.085
</td>
</tr>
</table>

> Same result!

``` r
sjPlot::plot_model(RLMM.Dur.I
                   , show.values=T
                   , show.p=T
                   , value.offset = 0.4
                   , value.size = 3.5
                   , wrap.title = 48
                   , title = "Effects of environment, presence of a sentinel, number of crows and bait on durations of behaviors")# Looks much better
```

![](MSC-R-Script_files/figure-gfm/Effect%20Sizes-1.png)<!-- --> \>Let’s
try the good old regular lmer and see if that changes anything.

    ## fixed-effect model matrix is rank deficient so dropping 2 columns / coefficients

    ## refitting model(s) with ML (instead of REML)

    ## Data: Crow
    ## Models:
    ## Simple.Dur.NoI: LDur ~ Behavior + Sentinel + CODED_NB_CROWS + Baited + G.Env + (1 | ID)
    ## Simple.Dur.I: LDur ~ Behavior + Sentinel * CODED_NB_CROWS * Baited + G.Env + (1 | ID)
    ##                npar    AIC    BIC  logLik deviance Chisq Df Pr(>Chisq)
    ## Simple.Dur.NoI    9 5589.3 5644.8 -2785.7   5571.3                    
    ## Simple.Dur.I     11 5593.0 5660.7 -2785.5   5571.0 0.378  2     0.8278

> Using the simple model, I looked at the AIC values. The model without
> interactions had the lowest AIC value, therefore is preferred.Let’s
> actually see what removing some factors in the model does to the AIC.

#### AIC Testing

    ## Single term deletions using Satterthwaite's method:
    ## 
    ## Model:
    ## LDur ~ Behavior + Sentinel + CODED_NB_CROWS + Baited + G.Env + (1 | ID)
    ##                 Sum Sq Mean Sq NumDF  DenDF F value  Pr(>F)    
    ## Behavior       22.8977 11.4488     2 3470.3 39.8790 < 2e-16 ***
    ## Sentinel        0.0424  0.0424     1  351.7  0.1477 0.70096    
    ## CODED_NB_CROWS  2.0979  2.0979     1   20.2  7.3076 0.01360 *  
    ## Baited          1.3249  1.3249     1   23.4  4.6150 0.04227 *  
    ## G.Env           0.2329  0.2329     1   16.7  0.8114 0.38053    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

> Seems like dropping any of the variables will have no effect on the
> model’s results. In other words, I can drop all factors except for
> ‘behavior’.

``` r
Simplest.Dur<-lmer(LDur~Behavior+(1|ID), data=Crow)
anova(Simple.Dur.NoI, Simplest.Dur)
```

    ## refitting model(s) with ML (instead of REML)

    ## Data: Crow
    ## Models:
    ## Simplest.Dur: LDur ~ Behavior + (1 | ID)
    ## Simple.Dur.NoI: LDur ~ Behavior + Sentinel + CODED_NB_CROWS + Baited + G.Env + (1 | ID)
    ##                npar    AIC    BIC  logLik deviance  Chisq Df Pr(>Chisq)  
    ## Simplest.Dur      5 5589.5 5620.3 -2789.8   5579.5                       
    ## Simple.Dur.NoI    9 5589.3 5644.8 -2785.7   5571.3 8.1813  4    0.08516 .
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

> Yep! The simplest model (LDur\~Behavio+(1\|ID)) is preferred. Let’s
> try subsetting the data.

#### Subset Data

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: LDur
    ##                 Chisq Df Pr(>Chisq)
    ## Sentinel       0.8612  1     0.3534
    ## CODED_NB_CROWS 2.0723  1     0.1500
    ## Baited         1.0262  1     0.3111
    ## G.Env          0.0379  1     0.8456

> Simple model on Head Up subset with no interactions shows no
> significance.

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: LDur
    ##                  Chisq Df Pr(>Chisq)    
    ## Sentinel        0.5011  1   0.479003    
    ## CODED_NB_CROWS 20.7974  1  5.105e-06 ***
    ## Baited          7.6112  1   0.005801 ** 
    ## G.Env           9.2813  1   0.002315 ** 
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

> Simple model on Head Down subset shows a significant effect of
> foraging group size, presence of bait and generalized environment.

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: LDur
    ##                 Chisq Df Pr(>Chisq)
    ## Sentinel       0.0084  1     0.9268
    ## CODED_NB_CROWS 0.2692  1     0.6039
    ## Baited         0.0975  1     0.7549
    ## G.Env          0.0085  1     0.9265

> Simple model shows no significant effects.

### Bout Summary

> Ok! Considering only the ‘Head Down’ subset had significant effects,
> let’s plot those results.

#### Bait

![](MSC-R-Script_files/figure-gfm/Bait%20effect%20on%20HD%20bout-1.png)<!-- -->

> Fun! The plot shows overlap between the standard error bars. In the
> presence of bait, crows will decrease the duration of their bouts of
> ‘Head Down’.

#### G.Env

![](MSC-R-Script_files/figure-gfm/G.Env%20effect%20on%20HD%20bout-1.png)<!-- -->

> Very nice! We can see that in Green Areas, the bout duration of head
> down behavior is increased. The error bars, as before, are standard
> error.

#### Foraging Group Size

![](MSC-R-Script_files/figure-gfm/NB_Crows%20effect%20on%20HD%20bout-1.png)<!-- -->

> Very nice! In larger foraging groups, individuals increase their bouts
> of head down behavior.

> I can therefore make the following conclusions: -Crows do not modify
> the duration of bouts of head up or moving -Crows will modify the
> duration of bouts of head down behavior. -In larger groups, crows will
> increase the duration of bouts of ‘Head Down”, possibly due to the
> many eyes hypothesis. -In the presence of Bait, crows will decrease
> the duration of bouts of ’Head Down’. This could be due to the larger
> concentration of food in the patch, and therefore less time required
> to look for food. -In Green Areas, crows will increase the duration of
> bouts of ‘Head Down’.

# Proportion Data

    ## 'data.frame':    252 obs. of  40 variables:
    ##  $ VIDEO_ID                         : Factor w/ 25 levels "037-2","038-2",..: 4 4 4 4 4 4 5 5 5 5 ...
    ##  $ ID                               : Factor w/ 67 levels "020-01-01","020-01-02",..: 1 1 1 2 2 2 3 3 3 3 ...
    ##  $ DATE                             : Factor w/ 19 levels "2022-06-25","2022-07-07",..: 1 1 1 1 1 1 2 2 2 2 ...
    ##  $ Convert.date                     : Factor w/ 19 levels "2022-06-25","2022-07-07",..: 1 1 1 1 1 1 2 2 2 2 ...
    ##  $ JULIAN_DATE                      : int  20227825 20227825 20227825 20227825 20227825 20227825 20227837 20227837 20227837 20227837 ...
    ##  $ TIME                             : Factor w/ 24 levels "6:13:00","6:18:00",..: 3 3 3 3 3 3 1 1 1 1 ...
    ##  $ DECIMAL_TIME                     : num  6.33 6.33 6.33 6.33 6.33 6.33 6.22 6.22 6.22 6.22 ...
    ##  $ LATITUDE                         : Factor w/ 11 levels "43°06'52.3\"N",..: 10 10 10 10 10 10 10 10 10 10 ...
    ##  $ LONGITUDE                        : Factor w/ 10 levels "79°06'48.3\"W",..: 8 8 8 8 8 8 8 8 8 8 ...
    ##  $ TEMPERATURE                      : int  18 18 18 18 18 18 16 16 16 16 ...
    ##  $ WEATHER                          : Factor w/ 6 levels "Cloudy","Foggy",..: 6 6 6 6 6 6 6 6 6 6 ...
    ##  $ TOTAL_VIDEO_DURATION             : num  87.7 87.7 87.7 87.7 87.7 ...
    ##  $ RECORDED_DURATION                : num  76.2 76.2 76.2 50 50 ...
    ##  $ CODED_ENV                        : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ GENERALIZED_ENVIRONMENT          : Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ CODED_SENTINEL_PRESENCE          : int  0 0 0 0 0 0 0 0 0 1 ...
    ##  $ SENTINEL_PRESENCE                : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 2 ...
    ##  $ CODED_BAIT_PRESENCE              : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ BAIT_PRESENCE                    : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ NUMBER_OF_CROWS_RECORDED         : int  2 2 2 2 2 2 2 2 2 2 ...
    ##  $ GROUP_SIZE                       : Factor w/ 2 levels "LARGE","SMALL": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ CODED_GROUP_SIZE                 : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ BEHAVIOR                         : Factor w/ 3 levels "HD","HU","M": 2 1 3 2 1 3 2 1 3 2 ...
    ##  $ CODED_BEHAVIOR                   : int  2 0 1 2 0 1 2 0 1 2 ...
    ##  $ NUMBER_OF_BOUTS                  : int  13 14 6 13 9 8 13 8 9 7 ...
    ##  $ BEHAVIOR_DURATION                : num  22.9 39.8 13.5 17.7 18.8 ...
    ##  $ MEAN_BOUT_DURATION               : num  1.76 2.84 2.25 1.36 2.09 ...
    ##  $ SD_BOUT_DURATION                 : num  2.74 2.44 2.2 1.3 1.17 ...
    ##  $ BEHAVIOR_PROPORTION              : num  0.301 0.522 0.177 0.354 0.375 0.27 0.68 0.209 0.111 0.779 ...
    ##  $ NUMBER_OF_PECKS                  : int  NA 31 NA NA 20 NA NA 9 NA NA ...
    ##  $ PECK_RATE                        : num  NA 46.8 NA NA 63.9 ...
    ##  $ TOTAL_NUMBER_OF_DISTURBANCES     : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ NUMBER_HUMAN_DISTURBANCE         : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ NUMBER_DOM_ANIMAL_DISTURBANCE    : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ NUMBER_HETEROSPECIFIC_DISTURBANCE: int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ NUMBER_VEHICLE_DISTURBANCE       : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ TOTAL_AGGRESSION                 : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ CONSPECIFIC_AGGRESSION           : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ TOTAL_FREQUENCY_OF_DISTURBANCES  : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ ASIN_PROPORTION                  : num  0.581 0.807 0.434 0.637 0.659 ...

> All variables are correctly categorized

## Stacked barplot for proportion

> I also decided to make a stacked barplot to show the proportion of
> total time spent performing each behavior.

    ##    GENERALIZED_ENVIRONMENT BEHAVIOR SENTINEL_PRESENCE  N BEHAVIOR_PROPORTION
    ## 1               Commercial       HD                NO 18           0.3391111
    ## 2               Commercial       HD               YES 31           0.3273548
    ## 3               Commercial       HU                NO 18           0.3870000
    ## 4               Commercial       HU               YES 31           0.3419355
    ## 5               Commercial        M                NO 18           0.2738889
    ## 6               Commercial        M               YES 31           0.3307097
    ## 7               Green Area       HD                NO 14           0.3827143
    ## 8               Green Area       HD               YES 21           0.3467619
    ## 9               Green Area       HU                NO 14           0.4247857
    ## 10              Green Area       HU               YES 21           0.3324286
    ## 11              Green Area        M                NO 14           0.1922857
    ## 12              Green Area        M               YES 21           0.3207143
    ##           sd         se         ci
    ## 1  0.2016388 0.04752673 0.10027264
    ## 2  0.1632008 0.02931173 0.05986254
    ## 3  0.1946321 0.04587522 0.09678826
    ## 4  0.1458862 0.02620194 0.05351151
    ## 5  0.1898621 0.04475094 0.09441622
    ## 6  0.2016228 0.03621252 0.07395583
    ## 7  0.1706122 0.04559803 0.09850856
    ## 8  0.1569611 0.03425172 0.07144784
    ## 9  0.1003549 0.02682097 0.05794319
    ## 10 0.1765697 0.03853067 0.08037356
    ## 11 0.1488424 0.03977981 0.08593905
    ## 12 0.2462048 0.05372630 0.11207109

> Summary values are good, but graphs are better. Let’s plot a stacked
> bar graph.

![](MSC-R-Script_files/figure-gfm/Proportion%20Barplot-1.png)<!-- -->

> The proportion figure shows something interesting: crows seem to spend
> longer times with their heads down in green areas than in commercial
> ones. Additionally, in the absence of a sentinel, individuals in green
> areas seemingly increased the proportion of time spent ‘alert’.  
>   
> This could be caused by food being easier to find in commercial areas.
> Crows in green areas need to spend more time looking for food. This
> reinforces the need to include “Baited” in the model. In the absence
> of a sentinel, individuals are expected to rely more on individual
> vigilance, explaining the apparent increase in the proportion of time
> spent alert in green areas with no sentinels.

## To transform, or not to transform?

> In order to determine whether our fixed effects have significant
> effects on the allocation of time to each behavior, I decided to first
> arcsin square root transform the proportions.

![](MSC-R-Script_files/figure-gfm/Proportion%20Histograms%20Combined-1.png)<!-- -->

> Transformed data seems more normal than untransformed data. Will
> continue to use transformed data.

## PCAs

> Let’s run another PCA

![](MSC-R-Script_files/figure-gfm/DATA%20PCA-1.png)<!-- -->![](MSC-R-Script_files/figure-gfm/DATA%20PCA-2.png)<!-- -->

> There are many variables here. I won’t bother describing them here.

> Since the presence of a sentinel is not “God-Given”, we’ll run another
> PCA without that factor.

![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20NO%20SENTINEL-1.png)<!-- -->![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20NO%20SENTINEL-2.png)<!-- -->

> Very similar to the one above.

### Scree Plot

    ##       eigenvalue variance.percent cumulative.variance.percent
    ## Dim.1 3.20998319        40.124790                    40.12479
    ## Dim.2 1.39393461        17.424183                    57.54897
    ## Dim.3 1.16158071        14.519759                    72.06873
    ## Dim.4 0.85186396        10.648299                    82.71703
    ## Dim.5 0.53938255         6.742282                    89.45931
    ## Dim.6 0.46716605         5.839576                    95.29889
    ## Dim.7 0.28823590         3.602949                    98.90184
    ## Dim.8 0.08785303         1.098163                   100.00000

![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20Scree%20Plot-1.png)<!-- -->

> The first PC covers \~40% of the variation. The first 4 PCs cover
> \~82% of the variation.

    ##       eigenvalue variance.percent cumulative.variance.percent
    ## Dim.1 3.05460025        43.637146                    43.63715
    ## Dim.2 1.32404519        18.914931                    62.55208
    ## Dim.3 1.11131041        15.875863                    78.42794
    ## Dim.4 0.56744231         8.106319                    86.53426
    ## Dim.5 0.50511508         7.215930                    93.75019
    ## Dim.6 0.34861692         4.980242                    98.73043
    ## Dim.7 0.08886983         1.269569                   100.00000

![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20Scree%20Plot%20NO%20SENTINEL-1.png)<!-- -->

> Excluding sentinel, the first PC covers \~43% of the variation, and
> the first 4 PCs cover \~86% of the total variation.

### Contribution of variables to Principal Components

![](MSC-R-Script_files/figure-gfm/PCA%20DATA%20Contribution%20Dim%201-1.png)<!-- -->

> Very weird, Julian Date, Decimal time, group size and generalized
> environment contribute greatly to PC1.

![](MSC-R-Script_files/figure-gfm/PCA%20DATA%20Contribution%20Dim%202-1.png)<!-- -->

> Frequency of disturbances, group size (again), temperature and the
> presence of a sentinel contribute most to PC2.

![](MSC-R-Script_files/figure-gfm/PCA%20DATA%20Contribution%20Dim%203-1.png)<!-- -->

> Temperature, Bait presence and decimal time contribute most to PC3.

![](MSC-R-Script_files/figure-gfm/PCA%20DATA%20Contribution%20Dim%204-1.png)<!-- -->

> Sentinel presence, environment and frequency of disturbances
> contribute most to PC4.

> I’m still a little unclear as to how I should interpret these results.

#### No Sentinel

![](MSC-R-Script_files/figure-gfm/PCA%20DATA.NOSENT%20Contribution%20Dim%201-1.png)<!-- -->

> Same 4 factors contribute the most to PC1 in the absence of the
> sentinel factor.

![](MSC-R-Script_files/figure-gfm/PCA%20DATA.NOSENT%20Contribution%20Dim%202-1.png)<!-- -->

> Same as above, frequency of disturbances, temperature and group size
> contribute most to PC2 in the absence of the sentinel factor.

![](MSC-R-Script_files/figure-gfm/PCA%20DATA.NOSENT%20Contribution%20Dim%203-1.png)<!-- -->

> Temperature, Bait presence and frequency of disturbances contribute
> the most to PC3 in the absence of a sentinel.

![](MSC-R-Script_files/figure-gfm/PCA%20DATA.NOSENT%20Contribution%20Dim%204-1.png)<!-- -->

> Environment contributes most to PC4 in the absence of a sentinel.

### BiPlots

![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20Biplot%20PC1-2-1.png)<!-- -->

> Messy, can’t see where the behaviors lie.

![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20Biplot%20PC1-4-1.png)<!-- -->

> Still messy.

![](MSC-R-Script_files/figure-gfm/DATA%20PCA%20Biplot%20PC2-3-1.png)<!-- -->

> Biplots are messy if we don’t include behavior.

> Same is observed if we run the biplot in the absence of the sentinel
> factor.

> I’m not sure how to interpret these results.

## Simple Model

> Let’s plot the simple model and test the assumptions. I’m using the
> same fixed effects as in the bout duration analyses. I am also adding
> some of the factors identified in the PCAs.

    ## boundary (singular) fit: see help('isSingular')

![](MSC-R-Script_files/figure-gfm/Proportion%20Simple%20Assumptions-1.png)<!-- -->

> Not great. The warning I got means my model could have collinearity in
> its fixed effects (two variables explaining the same thing). Let’s try
> dropping fixed effects one at a time.

> Let’s see the results before we do anything.

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: ASIN_PROPORTION
    ##                                  Chisq Df Pr(>Chisq)
    ## BEHAVIOR                        3.3906  2     0.1835
    ## SENTINEL_PRESENCE               0.1914  1     0.6617
    ## GENERALIZED_ENVIRONMENT         0.0710  1     0.7898
    ## BAIT_PRESENCE                   0.0139  1     0.9062
    ## GROUP_SIZE                      0.1681  1     0.6818
    ## TOTAL_FREQUENCY_OF_DISTURBANCES 0.0945  1     0.7585

    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')

    ## ANOVA-like table for random-effects: Single term deletions
    ## 
    ## Model:
    ## ASIN_PROPORTION ~ BEHAVIOR + SENTINEL_PRESENCE + GENERALIZED_ENVIRONMENT + BAIT_PRESENCE + GROUP_SIZE + TOTAL_FREQUENCY_OF_DISTURBANCES + (1 | ID) + (1 | JULIAN_DATE) + (1 | DECIMAL_TIME) + (1 | TEMPERATURE)
    ##                    npar  logLik    AIC LRT Df Pr(>Chisq)
    ## <none>               13 -17.093 60.185                  
    ## (1 | ID)             12 -17.093 58.185   0  1          1
    ## (1 | JULIAN_DATE)    12 -17.093 58.185   0  1          1
    ## (1 | DECIMAL_TIME)   12 -17.093 58.185   0  1          1
    ## (1 | TEMPERATURE)    12 -17.093 58.185   0  1          1

> Interesting. For one, the results are insignificant. The other, the
> ranova results suggest that the model without random effects is
> preferred.

``` r
drop1(Simple.Prop, test="Chisq")
```

    ## Single term deletions using Satterthwaite's method:
    ## 
    ## Model:
    ## ASIN_PROPORTION ~ BEHAVIOR + SENTINEL_PRESENCE + GENERALIZED_ENVIRONMENT + BAIT_PRESENCE + GROUP_SIZE + TOTAL_FREQUENCY_OF_DISTURBANCES + (1 | ID) + (1 | JULIAN_DATE) + (1 | DECIMAL_TIME) + (1 | TEMPERATURE)
    ##                                   Sum Sq  Mean Sq NumDF DenDF F value Pr(>F)
    ## BEHAVIOR                        0.198522 0.099261     2   244  1.6953 0.1857
    ## SENTINEL_PRESENCE               0.011208 0.011208     1   244  0.1914 0.6621
    ## GENERALIZED_ENVIRONMENT         0.004160 0.004160     1   244  0.0710 0.7900
    ## BAIT_PRESENCE                   0.000813 0.000813     1   244  0.0139 0.9063
    ## GROUP_SIZE                      0.009844 0.009844     1   244  0.1681 0.6821
    ## TOTAL_FREQUENCY_OF_DISTURBANCES 0.005534 0.005534     1   244  0.0945 0.7588

> Removing factors has no significant effects on the model’s output.

> Let’s go to an even simpler model

### The even simpler models

    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')
    ## boundary (singular) fit: see help('isSingular')

    ## refitting model(s) with ML (instead of REML)

    ## Data: DATA
    ## Models:
    ## Simpler.Prop5: ASIN_PROPORTION ~ BEHAVIOR + (1 | ID)
    ## Simpler.Prop4: ASIN_PROPORTION ~ BEHAVIOR + SENTINEL_PRESENCE + (1 | ID)
    ## Simpler.Prop3: ASIN_PROPORTION ~ BEHAVIOR + SENTINEL_PRESENCE + GENERALIZED_ENVIRONMENT + (1 | ID)
    ## Simpler.Prop2: ASIN_PROPORTION ~ BEHAVIOR + SENTINEL_PRESENCE + GENERALIZED_ENVIRONMENT + BAIT_PRESENCE + (1 | ID)
    ## Simpler.Prop1: ASIN_PROPORTION ~ BEHAVIOR + SENTINEL_PRESENCE + GENERALIZED_ENVIRONMENT + BAIT_PRESENCE + GROUP_SIZE + (1 | ID)
    ##               npar    AIC    BIC logLik deviance  Chisq Df Pr(>Chisq)
    ## Simpler.Prop5    5 2.1996 19.847 3.9002  -7.8004                     
    ## Simpler.Prop4    6 4.1442 25.321 3.9279  -7.8558 0.0554  1     0.8139
    ## Simpler.Prop3    7 6.1404 30.846 3.9298  -7.8596 0.0037  1     0.9512
    ## Simpler.Prop2    8 8.0820 36.317 3.9590  -7.9180 0.0585  1     0.8089
    ## Simpler.Prop1    9 9.9729 41.738 4.0136  -8.0271 0.1091  1     0.7412

> The simplest model, the one with only behavior as a FE and ID as a RE
> has the lowest AIC value. Weird…

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: ASIN_PROPORTION
    ##           Chisq Df Pr(>Chisq)
    ## BEHAVIOR 3.4556  2     0.1777

> Behavior is still insignificant. Could this be caused by the data
> transformation?

# Peck rate

> Using the peck data can allow us to infer if crows foraging
> effectiveness. More pecks per minute would imply that crows forage
> more actively. To analyse this, I used a linear mixed model with G.Env
> and presence of a sentinel and its interaction as fixed effects, and
> observation ID as the random effect.
>
> I calculated the pecks per minute of each individual recorded by
> dividing the number of pecks by the total head down duration (excluded
> time out of frame).

## Histogram & Transformation

> First, I made a histogram and decided if transformation was
> appropriate.

![](MSC-R-Script_files/figure-gfm/Peck%20Per%20Minute%20Histograms%20Combined-1.png)<!-- -->

> Some gaps are present in the histogram. Log transformation made the
> distribution more normal.  
> Let’s summarize the results:

## Peck Summary

    ##   GENERALIZED_ENVIRONMENT SENTINEL_PRESENCE  N PECK_RATE       sd       se
    ## 1              Commercial                NO 18  56.48511 24.21051 5.706472
    ## 2              Commercial               YES 31  72.94547 27.22061 4.888965
    ## 3              Green Area                NO 14  51.99745 24.32497 6.501122
    ## 4              Green Area               YES 21  46.76497 27.63062 6.029495
    ##          ci
    ## 1 12.039604
    ## 2  9.984599
    ## 3 14.044820
    ## 4 12.577307

> Again, summary stats are good, but plots are better. Let’s do that.
> Seems like peck rate is lower in the presence of a sentinel and in
> green areas. Standard error is relatively equal.

## Peck Plot

![](MSC-R-Script_files/figure-gfm/Peck%20Plots-1.png)<!-- -->

> It appears that there more pecks per minute in commercial areas in the
> presence of a sentinel.

## Simple Model

> At this point, let’s use the model. As before, we are using the
> presence of a sentinel, the type of environment and their interaction
> as fixed effects, with the trial ID as a random effect.

``` r
Anova(LMM.Peck)
```

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: LPek
    ##                                            Chisq Df Pr(>Chisq)   
    ## SENTINEL_PRESENCE                         0.0329  1   0.856044   
    ## GENERALIZED_ENVIRONMENT                   6.6910  1   0.009690 **
    ## BAIT_PRESENCE                             6.8827  1   0.008703 **
    ## SENTINEL_PRESENCE:GENERALIZED_ENVIRONMENT 1.4109  1   0.234906   
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
ranova(LMM.Peck)
```

    ## ANOVA-like table for random-effects: Single term deletions
    ## 
    ## Model:
    ## LPek ~ SENTINEL_PRESENCE + GENERALIZED_ENVIRONMENT + BAIT_PRESENCE + (1 | ID) + SENTINEL_PRESENCE:GENERALIZED_ENVIRONMENT
    ##          npar  logLik    AIC    LRT Df Pr(>Chisq)
    ## <none>      7 -35.218 84.435                     
    ## (1 | ID)    6 -35.829 83.658 1.2223  1     0.2689

> Generalized environment and bait presence are significant. Weirdly, it
> seems to also prefer the model without the random effect.

## Plot time

### Generalized Environment

![](MSC-R-Script_files/figure-gfm/G.Env%20effect%20on%20Peck%20Rate-1.png)<!-- -->

> Very cool! Peck rate is increased in commercial areas.

### Bait

![](MSC-R-Script_files/figure-gfm/Bait%20effect%20on%20Peck%20Rate-1.png)<!-- -->

> Seems like peck rate increases significantly in the presence of bait.
