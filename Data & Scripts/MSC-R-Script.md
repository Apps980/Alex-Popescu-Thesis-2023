R Script V4
================
Alex Popescu
2023-01-23

V4 - Made some revisions and added plots. All colors are colorblind
friendly.

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
> crows are expected to move less than non-baited crows. \#KMG: I can
> imagine a reviewer would want to know hwo the baiting was
> standardized - something to keep in mind when you are writing. I
> removed the observations in agricultural settings because they had no
> sentinels present. This could be talked about in the discussion:
> agricultural fields may lack perches for individuals to act as
> sentinels. Vineyards may have some structures, but the visibility is
> limited when the vines are laden with fruit.  
> \#KMG: This is something that can go in the methods. You had three
> sites, but one was removed due to lack of sentinals, and then you can
> have a bit in the discussion. I removed observations past Sept 29
> because I was told behaviors can change wildly in the winter. The
> number of crows spotted decreased past that date, suggesting the
> migrants have left or were leaving. To not have to include time of
> year (I already have part of the breeding season), I decided to
> simplify.  

### Data Management

> After combining the 6 tables into a long one, labeling the behaviors
> and the presence of a sentinel on the way, I made sure that the
> headers were factors.  

    ## 'data.frame':    3504 obs. of  9 variables:
    ##  $ Observation.id : Factor w/ 25 levels "020 - FVM - 2 crows - No Bait",..: 2 2 2 2 2 2 2 4 4 4 ...
    ##  $ Duration       : num  2.029 3.995 7.994 0.498 7.773 ...
    ##  $ ID             : Factor w/ 22 levels "20","24","25",..: 2 2 2 2 2 2 2 4 4 4 ...
    ##  $ G.Env          : Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Number.of.Crows: num  2 2 2 2 2 2 2 1 1 1 ...
    ##  $ Baited         : Factor w/ 2 levels "N","Y": 1 1 1 1 1 1 1 2 2 2 ...
    ##  $ Sentinel       : Factor w/ 2 levels "N","Y": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Behavior       : Factor w/ 3 levels "HD","HU","M": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ LDur           : num  0.708 1.385 2.079 -0.697 2.051 ...

> Good news! Observation ID, bait, presence of a sentinel and behavior
> are indeed recognized as factors! Number of crows and bout duration as
> a numeric variable.

# Bout Duration

## Figures

### Histogram

> I first decided to look at the distribution of my durations. I set up
> a histogram with the distibution of durations across all behaviors. I
> had to log-transform the duration to get a better distribution.  

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

> \#KMG: Is there a way to group the violin plot like the boxplot? So
> the violins for foraging sentinal present/absent are next to each
> other? Will be easier to see your conclusions. Also, have you looked
> at plotting means and SE? Here’s a plot \#Kiyoko plot of means and SE.
> KMG: I’ve put the function to generate means, SE, SD, and CI in the
> github. With this plot, it’s easier to see the differences in the
> commercial area where the sentinal presence increases the behaviours.
> It’s much messier in the green areas. I wonder if it’s a refuge or
> visibility thing?

![](MSC-R-Script_files/figure-gfm/Dot%20Plot%20V2-1.png)<!-- -->

> Very cool! I modified the violin and boxplot, but I think the last
> plot is the most informative. We can see that the presence of a
> sentinel increases the duration of bouts in commercial areas. In green
> areas, however, this effect is reduced apart from in alert behavior.

## The Tests

### Assumption testing first

> I used robustlmm::rlmer to create the models. It takes longer to run
> than the regular lmer test because of the computations it makes to
> weigh the model. I then plotted the models to test the assumptions of
> the model. The model I used was “log(Duration) \~ Behavior +
> (Sentinel\* G.Env) + Number of crows + Baited + (1\|ID)”. KMG: what is
> the backslash?
>
> Behavior is expected to be affected by both the presence of a sentinel
> and the environment the crows are in. I am therefore curious to see
> the interactions between these factors. The presence of bait and
> number of crows are not linked with the presence of a sentinel or the
> environment. Arguably, larger groups are more likely to have sentinels
> than not, but I don’t believe the interaction to be important here.

### RLMM

![](MSC-R-Script_files/figure-gfm/Duration%20RLMM-1.png)<!-- -->![](MSC-R-Script_files/figure-gfm/Duration%20RLMM-2.png)<!-- -->![](MSC-R-Script_files/figure-gfm/Duration%20RLMM-3.png)<!-- -->

> That took AGES to run. There’s a big “cliff” on the normal Q-Q vs
> residuals plot. This means that there are more extreme values than
> would be expected if they came from a truly normal population.

### Bout Results

    ## Robust linear mixed model fit by DAStau 
    ## Formula: LDur ~ Behavior + Sentinel * G.Env + Number.of.Crows + Baited +      (1 | ID) 
    ##    Data: Crow 
    ## 
    ## Scaled residuals: 
    ##     Min      1Q  Median      3Q     Max 
    ## -8.2780 -0.6589 -0.0230  0.6469  4.1819 
    ## 
    ## Random effects:
    ##  Groups   Name        Variance Std.Dev.
    ##  ID       (Intercept) 0.01112  0.1055  
    ##  Residual             0.81287  0.9016  
    ## Number of obs: 3504, groups: ID, 22
    ## 
    ## Fixed effects:
    ##                           Estimate Std. Error t value
    ## (Intercept)                0.33952    0.10117   3.356
    ## BehaviorHU                -0.43879    0.03602 -12.181
    ## BehaviorM                 -0.12055    0.04178  -2.886
    ## SentinelY                  0.18122    0.07719   2.348
    ## G.EnvGreen Area            0.25140    0.08371   3.003
    ## Number.of.Crows            0.02105    0.01593   1.321
    ## BaitedY                   -0.14686    0.06848  -2.145
    ## SentinelY:G.EnvGreen Area -0.29126    0.09573  -3.042
    ## 
    ## Correlation of Fixed Effects:
    ##             (Intr) BhvrHU BehvrM SntnlY G.EnGA Nmb..C BaitdY
    ## BehaviorHU  -0.195                                          
    ## BehaviorM   -0.197  0.464                                   
    ## SentinelY    0.050 -0.017 -0.004                            
    ## G.EnvGrenAr -0.626  0.006  0.015  0.245                     
    ## Nmbr.f.Crws -0.541 -0.001 -0.027 -0.411  0.355              
    ## BaitedY     -0.543  0.004  0.065 -0.240  0.044 -0.055       
    ## SntnY:G.EGA -0.042  0.015 -0.003 -0.781 -0.505  0.322  0.232
    ## 
    ## Robustness weights for the residuals: 
    ##  2745 weights are ~= 1. The remaining 759 ones are summarized as
    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##   0.162   0.698   0.836   0.788   0.912   0.999 
    ## 
    ## Robustness weights for the random effects: 
    ##  20 weights are ~= 1. The remaining 2 ones are
    ##     9    15 
    ## 0.993 0.619 
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
Effects of generalized environment, bait, number of crows and the
presence of a sentinel on durations of behaviors
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
0.34
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.14 – 0.54
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Behavior \[HU\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.44
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.51 – -0.37
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
-0.12
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.20 – -0.04
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
0.18
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.03 – 0.33
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.019</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
G Env \[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.25
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.09 – 0.42
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.003</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Number of Crows
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.02
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.01 – 0.05
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.186
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Baited \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.15
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.28 – -0.01
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.032</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\] × G Env<br>\[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.29
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.48 – -0.10
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>0.002</strong>
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
0.81
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
0.01
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
3504
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
Marginal R<sup>2</sup> / Conditional R<sup>2</sup>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.056 / 0.068
</td>
</tr>
</table>

> Like this! This is much better! Lets try a simpler output. Curious…
> Removing the three-way interaction caused many factors to become
> significant. Number of crows remains non-significant. \#KMG: Can we
> visualize the the baited effects on duration as well?

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: LDur
    ##                    Chisq Df Pr(>Chisq)    
    ## Behavior        113.8875  2    < 2e-16 ***
    ## Sentinel          0.3220  1    0.57040    
    ## G.Env             0.2304  1    0.63125    
    ## Number.of.Crows   0.0183  1    0.89237    
    ## Baited            1.0547  1    0.30443    
    ## Sentinel:G.Env    6.4054  1    0.01138 *  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    ## ANOVA-like table for random-effects: Single term deletions
    ## 
    ## Model:
    ## LDur ~ Behavior + Sentinel + G.Env + Number.of.Crows + Baited + (1 | ID) + Sentinel:G.Env
    ##          npar  logLik   AIC    LRT Df Pr(>Chisq)    
    ## <none>     10 -5055.7 10131                         
    ## (1 | ID)    9 -5072.3 10163 33.241  1   8.14e-09 ***
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

> Simple model shows significant effects of Behavior, and the
> interaction between generalized environment and Behavior, and
> generalized environment and the presence of a sentinel.

### Bout Summary

> I generated a plot to better see what the interaction between
> Generalized Environment and the presence of a sentinel. First, let’s
> make a summary of the results.

    ##         G.Env Behavior Sentinel   N Duration       sd         se        ci
    ## 1  Commercial       HD        N 278 1.613932 1.579565 0.09473603 0.1864940
    ## 2  Commercial       HD        Y 250 2.423120 2.939188 0.18589057 0.3661183
    ## 3  Commercial       HU        N 313 1.599853 2.546793 0.14395323 0.2832419
    ## 4  Commercial       HU        Y 327 2.293700 3.655008 0.20212243 0.3976289
    ## 5  Commercial        M        N 208 2.200827 2.512928 0.17424018 0.3435128
    ## 6  Commercial        M        Y 219 2.758694 3.388803 0.22899412 0.4513258
    ## 7  Green Area       HD        N 221 2.469910 2.808757 0.18893747 0.3723590
    ## 8  Green Area       HD        Y 482 2.259726 2.641513 0.12031764 0.2364131
    ## 9  Green Area       HU        N 250 2.211836 3.674204 0.23237704 0.4576751
    ## 10 Green Area       HU        Y 535 1.616030 2.799762 0.12104425 0.2377813
    ## 11 Green Area        M        N 113 1.943053 1.731280 0.16286512 0.3226964
    ## 12 Green Area        M        Y 308 2.008614 2.180730 0.12425864 0.2445064

> There are too many combinations to effectively see any trends, let’s
> look at the significant interactions from the models.

### Bait

![](MSC-R-Script_files/figure-gfm/Bait%20effect%20on%20bout-1.png)<!-- -->

### Plot the Interactions

<!-- --- -->
<!-- ```{r Behavior x G.Env Plot, echo = F} -->
<!-- BGInteraction<-Crow %>% -->
<!--   group_by(Behavior, G.Env) %>% -->
<!--   summarize(mean = mean(log(Duration)) -->
<!--             , se = sd(log(Duration))/sqrt(length(log(Duration))) -->
<!--             , upper = mean(log(Duration))+sd(log(Duration))/sqrt(length(log(Duration))) -->
<!--             , lower = mean(log(Duration))-sd(log(Duration))/sqrt(length(log(Duration))) -->
<!--             , .groups = 'keep') %>% -->
<!--   ggplot(aes(x = G.Env, y = mean, colour = Behavior))+ -->
<!--   geom_point(position = position_dodge(width = 0.9)) + -->
<!--   geom_text(aes(label = signif(mean)) -->
<!--             , size = 3 -->
<!--             , position=position_dodge2(width=0.9, preserve='single') -->
<!--             , hjust = 1.1 -->
<!--             , vjust = -.5)+ -->
<!--   geom_errorbar(aes(ymin=lower, ymax=upper) -->
<!--                 , width = 0.2 -->
<!--                 , position = position_dodge(width=.9))+ -->
<!--   theme_bw()+ -->
<!--   theme(text=element_text(size = 14))+ -->
<!--   labs(colour = "Behavior")+ -->
<!--   xlab("")+ -->
<!--   ylab("Mean log Bout Duration")+ -->
<!--   scale_colour_manual(values = c("orange", "blue", "green") -->
<!--                     , labels = c("Foraging", "Alert", "Moving"))+ -->
<!--   scale_x_discrete(labels=c("Commercial", "Green Area"))+ -->
<!--   theme(legend.position = 'bottom') -->
<!-- BGInteraction -->
<!-- ``` -->
<!-- > The error bars here represent the standard error. It looks like in both environments, the duration of bouts of alertness are smaller than the other two behaviors. Interestingly, in Green areas, individuals decrease the durations of bouts of movement and increase the duration of bouts of foraging. Let's next check out the interaction between the presence of a sentinel and generalized environment. -->

![](MSC-R-Script_files/figure-gfm/Sentinel%20x%20G.Env%20Plot-1.png)<!-- -->

> Very cool! I believe I’m getting better at this (or I make good
> annotations on other scripts…). There is a difference in bout duration
> between foragers in the presence and absence of a sentinel. The
> difference is reduced in green areas. Again, the error bars are the
> standard error. KMG: Cool. You’ll want to think about what exactly do
> you want to present. You can generate all sorts of figures. I
> personally think these means and SE plots are better. You can add the
> raw data in as a cloud of points, or you can add histograms sideways
> next to the plots as well.

### Effect Size

> I next plotted the effect size for each effect.

![](MSC-R-Script_files/figure-gfm/Duration%20Effect%20Size-1.png)<!-- -->

> Interesting… According to my model, Alert behavior had a significant
> negative effect on duration, while the presence of a sentinel and
> Green Areas have a significant positive effect on duration. The
> interaction between Moving and Green areas also had a significant
> negative effect on bout duration. I’m still relatively green when it
> comes to LMMs, so this may be evident, but I’ll make my assumptions
> clear. From the previous figure, I see that “Head Down”,
> G.Env\[Commercial\], Baited\[N\], etc are missing. Therefore I assume
> that the comparisons are made with those. \#KMG: THat is correct.
> Therefore, I can conclude that: - Crows spend significantly less time
> with their head’s up than down. - Crows spend significantly less time
> moving than with their heads down. - The duration of behaviors
> decreases significantly in the absence of a sentinel. - The duration
> of behaviors decreases significantly in Commercial areas. - The
> duration of behaviors decreases significantly in baited sites. KMG:
> When you present these kinds of things, its helpful to write in the
> same direction. So instead of significantly less and increases,
> perhaps significantly more and increases, or significantly less and
> decreases. - There was an interaction between the presence of a
> sentinel and the generalized environment. That being said, many papers
> start removing variables to fine tune their model. This is something I
> am considering in order to better identify the effects of the presence
> of a sentinel and the type of environment in which the individual
> forages in.

### What about the group size?

> From the previous results, the number of crows had little effect on
> the individual vigilance of foragers. This is slightly unexpected,
> since we would assume there is safety in numbers. By that logic, and
> hypotheses like the ‘many eyes’ hypothesis, we would expect to observe
> a decrease in the duration of some behaviors, namely “Alert”. To test
> this, I decided to plot the model’s estimates on the duration of
> behavior “Alert” to see if there is any trend.

    ## 
    ##  Number.of.Crows effect
    ## Number.of.Crows
    ##         1         3         6         8        10 
    ## 0.1815977 0.2237013 0.2868568 0.3289604 0.3710640 
    ## 
    ##  Lower 95 Percent Confidence Limits
    ## Number.of.Crows
    ##          1          3          6          8         10 
    ## 0.09029736 0.16331731 0.18080281 0.16749239 0.15037875 
    ## 
    ##  Upper 95 Percent Confidence Limits
    ## Number.of.Crows
    ##         1         3         6         8        10 
    ## 0.2728981 0.2840854 0.3929108 0.4904284 0.5917493

![](MSC-R-Script_files/figure-gfm/Effects%20of%20Group%20Size-1.png)<!-- -->

> Looks like the data does not follow the predictions from the model. I
> therefore conclude that the group size had no effect on the duration
> of alert behavior, at any group size (that I tested).

# Proportion Data

    ## 'data.frame':    99 obs. of  11 variables:
    ##  $ Observation.id : Factor w/ 25 levels "020 - FVM - 2 crows - No Bait",..: 1 1 1 2 2 2 2 2 2 3 ...
    ##  $ Behavior       : Factor w/ 3 levels "HD","HU","M": 1 2 3 1 1 2 2 3 3 1 ...
    ##  $ Sentinel       : Factor w/ 2 levels "N","Y": 1 1 1 1 2 1 2 1 2 1 ...
    ##  $ G.Env          : Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ Number.of.Crows: int  2 2 2 2 2 2 2 2 2 1 ...
    ##  $ Baited         : Factor w/ 2 levels "N","Y": 1 1 1 1 1 1 1 1 1 2 ...
    ##  $ sum            : num  46.02 36.19 25.27 29.63 4.22 ...
    ##  $ ID             : Factor w/ 22 levels "20","24","25",..: 1 1 1 2 2 2 2 2 2 3 ...
    ##  $ Total.Duration : num  107.5 107.5 107.5 131.2 39.6 ...
    ##  $ Proportion     : num  0.428 0.337 0.235 0.226 0.107 ...
    ##  $ AProp          : num  0.713 0.619 0.506 0.495 0.333 ...

> All variables are correctly categorized

## Stacked barplot for proportion

> I also decided to make a stacked barplot to show the proportion of
> total time spent performing each behavior.

    ##         G.Env Behavior Sentinel  N Proportion         sd         se         ci
    ## 1  Commercial       HD        N  6  0.3188697 0.09545968 0.03897125 0.10017879
    ## 2  Commercial       HD        Y  7  0.2873808 0.09427382 0.03563216 0.08718874
    ## 3  Commercial       HU        N  6  0.3830175 0.08976607 0.03664685 0.09420372
    ## 4  Commercial       HU        Y  7  0.3850826 0.09620465 0.03636194 0.08897446
    ## 5  Commercial        M        N  6  0.2981127 0.10783944 0.04402527 0.11317056
    ## 6  Commercial        M        Y  7  0.3275366 0.14721878 0.05564347 0.13615466
    ## 7  Green Area       HD        N  8  0.3942013 0.10500642 0.03712538 0.08778756
    ## 8  Green Area       HD        Y 12  0.3682277 0.11922277 0.03441665 0.07575054
    ## 9  Green Area       HU        N  8  0.4229948 0.10047222 0.03552229 0.08399688
    ## 10 Green Area       HU        Y 12  0.3865450 0.15717134 0.04537146 0.09986190
    ## 11 Green Area        M        N  8  0.1828039 0.07398395 0.02615728 0.06185213
    ## 12 Green Area        M        Y 12  0.2452273 0.09925096 0.02865129 0.06306105

> Summary values are good, but graphs are better. Let’s plot a stacked
> bar graph.

![](MSC-R-Script_files/figure-gfm/Proportion%20Barplot-1.png)<!-- -->

> The proportion figure shows something interesting: crows spent longer
> times with their heads down in green areas than in commercial ones.
> Additionally, in the absence of a sentinel, individuals in green areas
> seemingly increased the proportion of time spent ‘alert’.  
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

## Simple Model

> Let’s plot the simple model and test the assumptions.

    ## boundary (singular) fit: see help('isSingular')

![](MSC-R-Script_files/figure-gfm/Proportion%20Simple%20Assumptions-1.png)<!-- -->

``` r
isSingular(Simple.Prop)
```

    ## [1] TRUE

> Not great. This means my model could have collinearity in its fixed
> effects (two variables explaining the same thing). Let’s try dropping
> fixed effects one at a time.

``` r
drop1(Simple.Prop, test="Chisq")
```

    ## Single term deletions using Satterthwaite's method:
    ## 
    ## Model:
    ## AProp ~ Behavior + Sentinel * G.Env + Baited + (1 | ID)
    ##                 Sum Sq  Mean Sq NumDF DenDF F value    Pr(>F)    
    ## Behavior       0.42313 0.211567     2    92 12.1344 2.103e-05 ***
    ## Baited         0.00001 0.000009     1    92  0.0005    0.9816    
    ## Sentinel:G.Env 0.00020 0.000204     1    92  0.0117    0.9140    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

> Ah ha! Behavior completely explains the proportion of time allocated
> to each behavior, causing the singularity error to appear. Not sure
> where to proceed from here. One thing you can do is make different
> models and then do AIC on them to find the best fit model.

## AIC

``` r
Simple.Prop2 <- lmer(AProp~Behavior+Sentinel*G.Env+(1|ID), data = Proportion)
```

    ## boundary (singular) fit: see help('isSingular')

``` r
#Also, I would make a separate column for your transformed data in your dataframe. Doing the calculations inline with the functions can get confusing with all the () you have to keep track of.
Simple.Prop3 <- lmer(asin(sqrt(Proportion))~Behavior+Sentinel +(1|ID), data = Proportion)
```

    ## boundary (singular) fit: see help('isSingular')

``` r
Simple.Prop4 <- lmer(asin(sqrt(Proportion))~Behavior+G.Env +(1|ID), data = Proportion)
```

    ## boundary (singular) fit: see help('isSingular')

``` r
Simple.Prop5 <- lmer(asin(sqrt(Proportion))~Behavior+(1|ID), data = Proportion)
```

    ## boundary (singular) fit: see help('isSingular')

``` r
anova(Simple.Prop, Simple.Prop2, Simple.Prop3, Simple.Prop4, Simple.Prop5)
```

    ## refitting model(s) with ML (instead of REML)

    ## Warning in optwrap(optimizer, devfun, x@theta, lower = x@lower, calc.derivs =
    ## TRUE, : convergence code 3 from bobyqa: bobyqa -- a trust region step failed to
    ## reduce q

    ## Data: Proportion
    ## Models:
    ## Simple.Prop5: asin(sqrt(Proportion)) ~ Behavior + (1 | ID)
    ## Simple.Prop3: asin(sqrt(Proportion)) ~ Behavior + Sentinel + (1 | ID)
    ## Simple.Prop4: asin(sqrt(Proportion)) ~ Behavior + G.Env + (1 | ID)
    ## Simple.Prop2: AProp ~ Behavior + Sentinel * G.Env + (1 | ID)
    ## Simple.Prop: AProp ~ Behavior + Sentinel * G.Env + Baited + (1 | ID)
    ##              npar     AIC      BIC logLik deviance  Chisq Df Pr(>Chisq)
    ## Simple.Prop5    5 -117.17 -104.193 63.584  -127.17                     
    ## Simple.Prop3    6 -115.17  -99.598 63.584  -127.17 0.0001  1     0.9907
    ## Simple.Prop4    6 -115.17  -99.602 63.586  -127.17 0.0037  0           
    ## Simple.Prop2    8 -111.19  -90.425 63.593  -127.19 0.0138  2     0.9931
    ## Simple.Prop     9 -109.19  -85.831 63.593  -127.19 0.0006  1     0.9809

``` r
#Don't worry about the warning if you get it. I just have to change a parameter. Basically, the model with the lowest AIC is the one with only behaviour. So you can confidently say that the type of behaviour has an effect on the proportion of time in a behaviour, and nothing else matters. Think about what this means together with duration. Why things have an effect on duration but not proportion?
```

<!-- --- -->
<!-- ```{r Proportion Simple Model Results, include = F} -->
<!-- Anova(Simple.Prop) -->
<!-- ranova(Simple.Prop) -->
<!-- ``` -->
<!-- > Not sure how to interpret the last output. However, behavior has a significant effect. Let's use the robust LMM to fit the model. -->
<!-- ```{r Robust Proportion, include = F} -->
<!-- RLMM.Prop<-rlmer(asin(sqrt(Proportion))~Behavior*Sentinel*G.Env + (1|ID), data = Proportion) -->
<!-- plot(RLMM.Prop) -->
<!-- ``` -->
<!-- > Looks... ok... Let's see the output -->
<!-- ```{r Summary RLMM.Prop, include = F} -->
<!-- ##### Table of results ##### -->
<!-- sjPlot::tab_model(RLMM.Prop -->
<!--                   , show.re.var = T -->
<!--                   , title = "Effects of generalized environment, number of crows and the presence of a sentinel on the proportion of time" -->
<!--                   , dv.labels = " Effects on proportion") -->
<!-- ##### Effect Size ##### -->
<!-- sjPlot::plot_model(RLMM.Prop -->
<!--                    , show.values=T -->
<!--                    , show.p=T -->
<!--                    , value.offset = 0.5 -->
<!--                    , wrap.title = 70 -->
<!--                    , value.size = 2 -->
<!--                    , title = "Effects of generalized environment, number of crows and the presence of a sentinel on the proportion of time") -->
<!-- ``` -->
<!-- > Curious... Contrary to the simple model, the robust LMM does not identify behavior as a significant effect. The interaction between Moving and G.Env is significant. -->
<!-- --- -->

# Peck rate

> Using the peck data can allow us to infer if crows foraging
> effectiveness. More pecks per minute would imply that crows forage
> more actively. To analyse this, I used a linear mixed model with G.Env
> and presence of a sentinel and its interaction as fixed effects, and
> observation ID as the random effect.
>
> I calculated the pecks per minute of each individual recorded by
> dividing the number of pecks by the total recorded duration (excluded
> time out of frame).

    ## 'data.frame':    33 obs. of  10 variables:
    ##  $ Observation.id      : Factor w/ 25 levels "020 - FVM - 2 crows - No Bait",..: 1 2 3 5 7 10 11 12 14 15 ...
    ##  $ Number.of.occurences: int  51 12 37 38 116 220 429 18 120 55 ...
    ##  $ ID                  : Factor w/ 22 levels "20","24","25",..: 1 2 3 5 7 10 11 12 14 15 ...
    ##  $ G.Env               : Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 1 1 1 2 1 ...
    ##  $ Number.of.Crows     : Factor w/ 7 levels "1","2","3","4",..: 2 2 1 1 2 4 2 1 2 6 ...
    ##  $ Baited              : Factor w/ 2 levels "N","Y": 1 1 2 2 2 1 2 2 2 2 ...
    ##  $ Sentinel            : Factor w/ 2 levels "N","Y": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ Total.Duration..s.  : num  107.5 131.2 83.4 140.2 290.5 ...
    ##  $ PPM                 : num  28.47 5.49 26.6 16.26 23.96 ...
    ##  $ LPek                : num  3.35 1.7 3.28 2.79 3.18 ...

> Just making sure! Seems like everything is ok.

## Histogram & Transformation

> First, I made a histogram and decided if transformation was
> appropriate.

![](MSC-R-Script_files/figure-gfm/Peck%20Per%20Minute%20Histograms%20Combined-1.png)<!-- -->

> Some gaps are present in the histogram. Log transformation made the
> distribution more normal.  
> Let’s summarize the results:

## Peck Summary

    ##        G.Env Sentinel  N      PPM       sd       se        ci
    ## 1 Commercial        N  6 34.38364 21.66762 8.845771 22.738777
    ## 2 Commercial        Y  7 29.26907 20.03885 7.573974 18.532846
    ## 3 Green Area        N  8 26.29041 13.45621 4.757487 11.249670
    ## 4 Green Area        Y 12 21.35999 13.53876 3.908304  8.602119

> Again, summary stats are good, but plots are better. Let’s do that.
> Seems like peck rate is lower in the presence of a sentinel and in
> green areas. Standard error is relatively equal.

## Peck Plot

![](MSC-R-Script_files/figure-gfm/Peck%20Plots-1.png)<!-- -->

> It appears that there are fewer pecks per minute in Green areas,
> regardless of the presence of a sentinel.

## Simple Model

> At this point, let’s use the model. As before, we are using the
> presence of a sentinel, the type of environment and their interaction
> as fixed effects, with the trial ID as a random effect.

![](MSC-R-Script_files/figure-gfm/Simple%20peck-1.png)<!-- -->

> Uhh… not good. Need to figure out why.Let’s try the robust version of
> the linear mixed model.

## Robust LMM

![](MSC-R-Script_files/figure-gfm/Robust%20Peck-1.png)<!-- -->![](MSC-R-Script_files/figure-gfm/Robust%20Peck-2.png)<!-- -->![](MSC-R-Script_files/figure-gfm/Robust%20Peck-3.png)<!-- -->

> First graph is identical, others seem ok.

``` r
Anova(LMM.Peck)
```

    ## Analysis of Deviance Table (Type II Wald chisquare tests)
    ## 
    ## Response: LPek
    ##                 Chisq Df Pr(>Chisq)  
    ## Sentinel       3.0943  1    0.07857 .
    ## G.Env          0.1121  1    0.73773  
    ## Baited         1.6241  1    0.20252  
    ## Sentinel:G.Env 0.0617  1    0.80376  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

``` r
ranova(LMM.Peck)
```

    ## ANOVA-like table for random-effects: Single term deletions
    ## 
    ## Model:
    ## LPek ~ Sentinel + G.Env + Baited + (1 | ID) + Sentinel:G.Env
    ##          npar  logLik    AIC    LRT Df Pr(>Chisq)
    ## <none>      7 -34.606 83.211                     
    ## (1 | ID)    6 -35.532 83.064 1.8529  1     0.1734

> All not significant, including random effect. Presence of a sentinel
> is marginally insignificant (p.val = 0.05419). Let’s try the robust
> model.

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">
Effects of generalized environment, bait, and the presence of a sentinel
on the peck rate of foraging crows
</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">
 
</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">
Effects on log of pecks per minute
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
3.14
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
2.42 – 3.85
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
<strong>\<0.001</strong>
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.35
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.80 – 0.10
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.125
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
G Env \[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.05
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.54 – 0.65
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.861
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Baited \[Y\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.24
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.29 – 0.77
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.382
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">
Sentinel \[Y\] × G Env<br>\[Green Area\]
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.15
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
-0.76 – 0.46
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">
0.627
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
0.12
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
τ<sub>00</sub> <sub>ID</sub>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.26
</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
ICC
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.70
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
33
</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">
Marginal R<sup>2</sup> / Conditional R<sup>2</sup>
</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">
0.142 / 0.739
</td>
</tr>
</table>

> Interesting! Looks like there are no effects on peck rate. Let’s plot
> it. The difference between the robust LMM and the simpler model is
> likely due to how the p-values are estimated, and how the robust lmm
> decreases the chance of committing Type I error.

``` r
sjPlot::plot_model(RLMM.Peck
                   , show.values=T
                   , show.p=T
                   , value.offset = 0.25
                   , value.size = 4
                   , wrap.title = 70
                   , title = "Effects of generalized environment, bait, and the presence of a sentinel on the peck rate of foraging crows")
```

![](MSC-R-Script_files/figure-gfm/Robust%20Peck%20Effect%20size-1.png)<!-- -->

KMG: Great job! I agree with your methods and conclusions. Looks like
you are pretty much done with your analysis! The hard part now will be
interpreting the results! For results, my suggestion is to stick with
plots of effect size and means w/ SE. Make sure your stats methods
parallels your results section, and in the resuts section present
everything in the same way/order. It might seem redundant, but it’s
actually a lot harder to understand if things are in all directions. I
think I commented on this above. I have tried to put all my comments
with a KMG: in front. I might have forgotten a few. You can check the
two most recent histories to see what changes I made.

# Disturbances

To determine whether there were more or fewer disturbances in different
areas, I decided to extract the number of disturbances from across my
videos, then compare across the same factors.
