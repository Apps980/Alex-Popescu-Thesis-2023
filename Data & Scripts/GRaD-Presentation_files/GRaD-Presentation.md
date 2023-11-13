---
title: "GRaD Presentation"
author: "Alex Popescu"
date: "2023-09-22"
output: 
  html_document: 
    keep_md: yes
---

# Bookkeeping






```r
labs.prop<-c(
  '(Intercept)' = "Intercept"
  , BEHAVIORHU = "Behavior"
  , 'GENERALIZED_ENVIRONMENTGreen Area' = "Generalized Environment"
  , GROUP_SIZESMALL = "Group Size"
  , BAIT_PRESENCEYES = "Bait Presence"
  , DISTURBANCE_FREQUENCY = "Disturbance Frequency"
  )

labs.bout.1<-c(
  '(Intercept)' = "Intercept"
  , BEHAVIORHU = "Behavior"
  , 'GENERALIZED_ENVIRONMENTGreen Area' = "Generalized Environment"
  , GROUP_SIZESMALL = "Group Size"
  , BAIT_PRESENCEYES = "Bait Presence"
  , DISTURBANCE_FREQUENCY = "Disturbance Frequency"
  , 'BEHAVIORHU:GENERALIZED_ENVIRONMENTGreen Area' = "Behavior: Generalized Environment"
)

labs.bout.2<-c(
  '(Intercept)' = "Intercept"
  , 'GENERALIZED_ENVIRONMENTGreen Area' = "Generalized Environment"
  , GROUP_SIZESMALL = "Group Size"
  , BAIT_PRESENCEYES = "Bait Presence"
  , DISTURBANCE_FREQUENCY = "Disturbance Frequency"
  )

labs.peck<-c(
  '(Intercept)' = "Intercept"
  , 'GENERALIZED_ENVIRONMENTGreen Area' = "Generalized Environment"
  , GROUP_SIZESMALL = "Group Size"
  , BAIT_PRESENCEYES = "Bait Presence"
  , DISTURBANCE_FREQUENCY = "Disturbance Frequency"
  , 'GENERALIZED_ENVIRONMENTGreen Area:DISTURBANCE_FREQUENCY' = "Generalized Environment: Disturbance Frequency"
  )
```


```r
DATA.SR<-read.csv("DATA.SR.csv", stringsAsFactors = T) %>%
  rename("VIDEO_ID" = "VIDEO_ID."
         , "ID" = "ID.")
BOUT.raw<-read.csv("BOUT.csv", stringsAsFactors = T)
```

# Proportion Data


```r
PROP<-DATA.SR[,c(1,2,15,17,19,22,30,36,41,46)]%>% 
  subset(.
         , HU_BEHAVIOR_PROPORTION_... != 0 
         & HD_BEHAVIOR_PROPORTION_... != 0 
         & M_BEHAVIOR_PROPORTION_... != 0
         ) %>% #Remove cases where proportion = 1 or 0
  rename("HU" = "HU_BEHAVIOR_PROPORTION_..."
         , "HD" = "HD_BEHAVIOR_PROPORTION_..."
         , "M" = "M_BEHAVIOR_PROPORTION_..."
         , "DISTURBANCE_FREQUENCY" = "TOTAL_FREQUENCY_OF_DISTURBANCES"
         ) %>%
  pivot_longer(., cols = c("HU"
                              , "HD"
                              , "M")
               , names_to = "BEHAVIOR"
               , values_to = "PROPORTION")
PROP$ASIN_PROPORTION<-asin(sqrt(PROP$PROPORTION)) #Transform for normality

str(PROP)
```

```
## tibble [228 × 10] (S3: tbl_df/tbl/data.frame)
##  $ VIDEO_ID               : Factor w/ 25 levels "037-2","038-2",..: 4 4 4 4 4 4 5 5 5 5 ...
##  $ ID                     : Factor w/ 64 levels "020-01-01","020-01-02",..: 1 1 1 2 2 2 3 3 3 3 ...
##  $ GENERALIZED_ENVIRONMENT: Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
##  $ SENTINEL_PRESENCE      : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 2 ...
##  $ BAIT_PRESENCE          : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 1 ...
##  $ GROUP_SIZE             : Factor w/ 2 levels "LARGE","SMALL": 2 2 2 2 2 2 2 2 2 2 ...
##  $ DISTURBANCE_FREQUENCY  : num [1:228] 0 0 0 0 0 0 0 0 0 0 ...
##  $ BEHAVIOR               : chr [1:228] "HU" "HD" "M" "HU" ...
##  $ PROPORTION             : num [1:228] 0.301 0.522 0.177 0.354 0.375 ...
##  $ ASIN_PROPORTION        : num [1:228] 0.581 0.807 0.434 0.638 0.659 ...
```


```r
PROP.SUMMARY <- summarySE(data = PROP
                      , measurevar = "PROPORTION"
                      , groupvars = c("GENERALIZED_ENVIRONMENT"
                                      , "BEHAVIOR"
                                      )
                      )
PROP.SUMMARY
```

```
## # A tibble: 6 × 7
##   GENERALIZED_ENVIRONMENT BEHAVIOR     N PROPORTION    sd     se     ci
##   <fct>                   <chr>    <dbl>      <dbl> <dbl>  <dbl>  <dbl>
## 1 Commercial              HD          44      0.340 0.153 0.0231 0.0466
## 2 Commercial              HU          44      0.369 0.115 0.0173 0.0349
## 3 Commercial              M           44      0.292 0.166 0.0250 0.0503
## 4 Green Area              HD          32      0.395 0.121 0.0214 0.0437
## 5 Green Area              HU          32      0.390 0.131 0.0232 0.0474
## 6 Green Area              M           32      0.215 0.112 0.0198 0.0403
```

## Stacked barplot


```r
PROP.BARPLOT<-ggplot(PROP.SUMMARY
                 , aes(x = GENERALIZED_ENVIRONMENT
                       , y = PROPORTION
                       , fill = BEHAVIOR))+
  geom_bar(stat = 'identity'
           , position = 'stack')+
  geom_text(aes(label = paste0(formattable::digits(PROPORTION*100, dig=2)
                               , "%"))
            , position = position_stack(vjust = 0.5)
            , size = 4) +
  scale_y_continuous(labels = scales::percent) +
  theme_classic()+
  ylab("Proportion of time")+
  scale_x_discrete(labels=c("Commercial Area"
                            , "Green Area"))+
  scale_fill_manual(values = cbPalette, labels = c("Foraging"
                                                   , "Alert"
                                                   , "Moving")
                    , name="")+
  theme(legend.position = "bottom"
        , text = element_text(size = 18)
        , axis.title.x = element_blank())
PROP.BARPLOT
```

![](GRaD-Presentation_files/figure-html/PROP Barplot-1.png)<!-- -->

## PROP Model


```r
PROP.MOD<-lm(ASIN_PROPORTION~BEHAVIOR+GENERALIZED_ENVIRONMENT+DISTURBANCE_FREQUENCY+BAIT_PRESENCE+GROUP_SIZE, data = PROP)

sjPlot::tab_model(PROP.MOD
                  , pred.labels = labs.prop
                  , rm.terms = "BEHAVIORM"
                  , show.re.var = T
                  , title = "Proportion Model 1 Output"
                  , dv.labels = " Effects of on the proportion of behaviors")
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">Proportion Model 1 Output</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; "> Effects of on the proportion of behaviors</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Estimates</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Intercept</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.65</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.57&nbsp;&ndash;&nbsp;0.72</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Behavior</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.03&nbsp;&ndash;&nbsp;0.07</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.496</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Generalized Environment</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.05&nbsp;&ndash;&nbsp;0.05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.989</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Disturbance Frequency</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.02&nbsp;&ndash;&nbsp;0.02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.947</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Bait Presence</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.01</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.07&nbsp;&ndash;&nbsp;0.05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.815</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Group Size</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.06&nbsp;&ndash;&nbsp;0.05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.920</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">228</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">R<sup>2</sup> / R<sup>2</sup> adjusted</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.135 / 0.112</td>
</tr>

</table>

Cool, nothing is significant (apart from "Moving" being different, but thats to be expected). 

Crows allocate similar time to foraging and vigilance.

I tinkered with the models, and the model with the lowest AIC value had no interactions and no random effects.

# Bout Data


```r
BOUT<-BOUT.raw %>%
  filter(.
         , DURATION > 0.01) %>% #Remove impossibly small values
  filter(.
         , BEHAVIOR != "M") #Remove "M"
BOUT$LDURATION<-log(BOUT$DURATION) #Transform data for normality
str(BOUT)
```

```
## 'data.frame':	3897 obs. of  16 variables:
##  $ VIDEO_ID               : Factor w/ 25 levels "037-2","038-2",..: 4 4 4 4 4 4 4 4 4 4 ...
##  $ ID                     : Factor w/ 67 levels "020-01-01","020-01-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ JULIAN_DATE            : int  20227825 20227825 20227825 20227825 20227825 20227825 20227825 20227825 20227825 20227825 ...
##  $ DECIMAL_TIME           : num  6.33 6.33 6.33 6.33 6.33 ...
##  $ LATITUDE               : num  43.2 43.2 43.2 43.2 43.2 ...
##  $ LONGITUDE              : num  -79.2 -79.2 -79.2 -79.2 -79.2 ...
##  $ TEMPERATURE            : int  18 18 18 18 18 18 18 18 18 18 ...
##  $ WEATHER                : Factor w/ 6 levels "Cloudy","Foggy",..: 6 6 6 6 6 6 6 6 6 6 ...
##  $ GROUP_SIZE             : Factor w/ 2 levels "LARGE","SMALL": 2 2 2 2 2 2 2 2 2 2 ...
##  $ BAIT_PRESENCE          : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 1 ...
##  $ GENERALIZED_ENVIRONMENT: Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
##  $ SENTINEL_PRESENCE      : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 1 ...
##  $ DISTURBANCE_FREQUENCY  : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BEHAVIOR               : Factor w/ 3 levels "HD","HU","M": 1 1 1 1 1 1 1 1 1 1 ...
##  $ DURATION               : num  2.38 2.51 3.51 1.75 10.5 ...
##  $ LDURATION              : num  0.869 0.919 1.255 0.56 2.351 ...
```

```r
str(BOUT.raw)
```

```
## 'data.frame':	5091 obs. of  15 variables:
##  $ VIDEO_ID               : Factor w/ 25 levels "037-2","038-2",..: 4 4 4 4 4 4 4 4 4 4 ...
##  $ ID                     : Factor w/ 67 levels "020-01-01","020-01-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ JULIAN_DATE            : int  20227825 20227825 20227825 20227825 20227825 20227825 20227825 20227825 20227825 20227825 ...
##  $ DECIMAL_TIME           : num  6.33 6.33 6.33 6.33 6.33 ...
##  $ LATITUDE               : num  43.2 43.2 43.2 43.2 43.2 ...
##  $ LONGITUDE              : num  -79.2 -79.2 -79.2 -79.2 -79.2 ...
##  $ TEMPERATURE            : int  18 18 18 18 18 18 18 18 18 18 ...
##  $ WEATHER                : Factor w/ 6 levels "Cloudy","Foggy",..: 6 6 6 6 6 6 6 6 6 6 ...
##  $ GROUP_SIZE             : Factor w/ 2 levels "LARGE","SMALL": 2 2 2 2 2 2 2 2 2 2 ...
##  $ BAIT_PRESENCE          : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 1 ...
##  $ GENERALIZED_ENVIRONMENT: Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
##  $ SENTINEL_PRESENCE      : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 1 1 1 1 1 ...
##  $ DISTURBANCE_FREQUENCY  : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ BEHAVIOR               : Factor w/ 3 levels "HD","HU","M": 1 1 1 1 1 1 1 1 1 1 ...
##  $ DURATION               : num  2.38 2.51 3.51 1.75 10.5 ...
```


```r
BOUT.SUMMARY <- summarySE(data = BOUT
                      , measurevar = "DURATION"
                      , groupvars = c("GENERALIZED_ENVIRONMENT"
                                      , "BEHAVIOR"
                                      )
                      ) %>%
  mutate(BEHAVIOR = recode_factor(BEHAVIOR
                                  ,'HD' = 'Foraging'#Rename HD as 'foraging'
                                  , 'HU' = 'Alert') #Rename HU as 'Alert'
                           )

BOUT.SUMMARY
```

```
##   GENERALIZED_ENVIRONMENT BEHAVIOR    N DURATION       sd         se         ci
## 1              Commercial Foraging  839 1.656611 1.232516 0.04255117 0.08351938
## 2              Commercial    Alert 1049 1.640384 1.972984 0.06091662 0.11953242
## 3              Green Area Foraging  948 2.086657 1.911171 0.06207193 0.12181444
## 4              Green Area    Alert 1061 1.640139 2.356629 0.07234916 0.14196384
```

## BOUT Dot Plot


```r
BOUT.DOTPLOT<-BOUT.SUMMARY %>%
           ggplot(.
               , aes(x = GENERALIZED_ENVIRONMENT
                     , y = DURATION
                     , colour = GENERALIZED_ENVIRONMENT))+
  geom_point(position = position_dodge(width = 0.9)
             , size = 3) +
  geom_errorbar(aes(ymin=(DURATION-se)
                    , ymax=(DURATION+se))
                , width = 0.1
                , position = position_dodge(width=0.9))+
  geom_text(aes(label = round(DURATION, 2)), hjust = -0.2, size = 6) +
  theme_classic() +
  ylab("Mean Bout Duration (s)") +
  scale_colour_manual(values = cbPalette
                      , guide="none") +
  scale_x_discrete(labels = c("Commercial Area", "Green Area"))+
  theme(axis.title.x = element_blank()
        , legend.position = "bottom"
        , legend.box="vertical"
        , legend.margin=margin()
        , text = element_text(size = 18)) +
  facet_grid(~BEHAVIOR)

BOUT.DOTPLOT
```

![](GRaD-Presentation_files/figure-html/BOUT Dot Plot-1.png)<!-- -->

> The error bars represent the standard error, while the symbols are the mean duration of bouts.

## BOUT Model - All behaviors


```r
BOUT.MOD<-rlmer(LDURATION~BEHAVIOR*GENERALIZED_ENVIRONMENT+GROUP_SIZE+BAIT_PRESENCE+DISTURBANCE_FREQUENCY+(1|VIDEO_ID/ID), data = BOUT)

sjPlot::tab_model(BOUT.MOD
                  , pred.labels = labs.bout.1
                  , show.re.var = T
                  , title = "Bout Robust Model  Output"
                  , dv.labels = " Effects on duration of bouts of all behaviors")
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">Bout Robust Model  Output</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; "> Effects on duration of bouts of all behaviors</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Estimates</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Intercept</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.45</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.27&nbsp;&ndash;&nbsp;0.63</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Behavior</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.30</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.38&nbsp;&ndash;&nbsp;-0.23</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Generalized Environment</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.22</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.08&nbsp;&ndash;&nbsp;0.36</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.002</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Group Size</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.19&nbsp;&ndash;&nbsp;0.08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.431</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Bait Presence</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.11</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.26&nbsp;&ndash;&nbsp;0.03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.130</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Disturbance Frequency</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.08</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.14&nbsp;&ndash;&nbsp;-0.02</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.005</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Behavior: Generalized<br>Environment</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.21</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.32&nbsp;&ndash;&nbsp;-0.11</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td colspan="4" style="font-weight:bold; text-align:left; padding-top:.8em;">Random Effects</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&sigma;<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.65</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>ID:VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.03</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.00</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">ICC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.04</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">64</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">25</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">3897</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">Marginal R<sup>2</sup> / Conditional R<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.072 / 0.110</td>
</tr>

</table>

```r
sjPlot::plot_model(BOUT.MOD
                  , axis.labels = labs.bout.1
                  , show.values=T
                  , show.p=T
                  , value.offset = 0.4
                  , value.size = 3.5
                  , wrap.title = 48
                  , show.re.var = T
                  , title = "Effects on duration of bouts of all behaviors"
                  )
```

![](GRaD-Presentation_files/figure-html/BOUT Model - All behaviors-1.png)<!-- -->

##BOUT model - Head Down


```r
BOUT.MOD.HD<-rlmer(LDURATION~GENERALIZED_ENVIRONMENT+GROUP_SIZE+BAIT_PRESENCE+DISTURBANCE_FREQUENCY+(1|VIDEO_ID/ID), data = subset(BOUT, BEHAVIOR == "HD"))
```

```
## boundary (singular) fit: see help('isSingular')
```

```r
sjPlot::tab_model(BOUT.MOD.HD
                  , pred.labels = labs.bout.2
                  , show.re.var = T
                  , title = "Head Down"
                  , dv.labels = " Effects on foraging bout duration")
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">Head Down</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; "> Effects on foraging bout duration</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Estimates</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Intercept</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.47</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.30&nbsp;&ndash;&nbsp;0.63</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Generalized Environment</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.24</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.13&nbsp;&ndash;&nbsp;0.35</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Group Size</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.16</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.28&nbsp;&ndash;&nbsp;-0.03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.013</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Bait Presence</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.11</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.25&nbsp;&ndash;&nbsp;0.04</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.142</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Disturbance Frequency</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.11</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.17&nbsp;&ndash;&nbsp;-0.05</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td colspan="4" style="font-weight:bold; text-align:left; padding-top:.8em;">Random Effects</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&sigma;<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.45</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>ID:VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.02</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.00</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">ICC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.03</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">64</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">25</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">1787</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">Marginal R<sup>2</sup> / Conditional R<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.047 / 0.079</td>
</tr>

</table>

```r
sjPlot::plot_model(BOUT.MOD.HD
                  , axis.labels = labs.bout.2
                  , show.values=T
                  , show.p=T
                  , value.offset = 0.4
                  , value.size = 3.5
                  , wrap.title = 48
                  , show.re.var = T
                  , title = "Effects on foraging bout duration")
```

![](GRaD-Presentation_files/figure-html/BOUT Model - Head Down-1.png)<!-- -->


```r
BOUT.MOD.HU<-rlmer(LDURATION~GENERALIZED_ENVIRONMENT+GROUP_SIZE+BAIT_PRESENCE+DISTURBANCE_FREQUENCY+(1|VIDEO_ID/ID), data = subset(BOUT, BEHAVIOR == "HU"))


sjPlot::tab_model(BOUT.MOD.HU
                  , pred.labels = labs.bout.2
                  , show.re.var = T
                  , title = "Head Up"
                  , dv.labels = " Effects on alert bout duration")
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">Head Up</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; "> Effects on alert bout duration</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Estimates</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Intercept</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.09</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.19&nbsp;&ndash;&nbsp;0.37</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.523</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Generalized Environment</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.00</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.19&nbsp;&ndash;&nbsp;0.20</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.965</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Group Size</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.07</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.14&nbsp;&ndash;&nbsp;0.29</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.506</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Bait Presence</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.10</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.34&nbsp;&ndash;&nbsp;0.13</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.394</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Disturbance Frequency</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.06</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;0.14&nbsp;&ndash;&nbsp;0.03</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.212</td>
</tr>
<tr>
<td colspan="4" style="font-weight:bold; text-align:left; padding-top:.8em;">Random Effects</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&sigma;<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.81</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>ID:VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.07</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.00</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">ICC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.08</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">63</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">25</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">2110</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">Marginal R<sup>2</sup> / Conditional R<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.005 / 0.089</td>
</tr>

</table>

```r
sjPlot::plot_model(BOUT.MOD.HU
                  , axis.labels = labs.bout.2
                  , show.values=T
                  , show.p=T
                  , value.offset = 0.4
                  , value.size = 3.5
                  , wrap.title = 48
                  , show.re.var = T
                  , title = "Effects on alert bout duration")
```

![](GRaD-Presentation_files/figure-html/BOUT Model - Head Up-1.png)<!-- -->

##BOUT Post Hoc


```r
BOUT.DIFF<-emmeans(BOUT.MOD, ~BEHAVIOR+GENERALIZED_ENVIRONMENT, pbkrtest.limit = 5070)
test(pairs(BOUT.DIFF, by= "BEHAVIOR"), adjust="fdr")
```

```
## BEHAVIOR = HD:
##  contrast                estimate     SE  df z.ratio p.value
##  Commercial - Green Area -0.22090 0.0701 Inf  -3.152  0.0016
## 
## BEHAVIOR = HU:
##  contrast                estimate     SE  df z.ratio p.value
##  Commercial - Green Area -0.00968 0.0678 Inf  -0.143  0.8864
## 
## Results are averaged over the levels of: GROUP_SIZE, BAIT_PRESENCE
```

#Peck Rate


```r
PECK<-DATA.SR[,c(1,2,15,17,19,22,30,38,48)] %>%
  subset(., HD_BEHAVIOR_DURATION > 0) %>%
  rename("DISTURBANCE_FREQUENCY" = "TOTAL_FREQUENCY_OF_DISTURBANCES")
  
str(PECK)
```

```
## 'data.frame':	79 obs. of  9 variables:
##  $ VIDEO_ID               : Factor w/ 25 levels "037-2","038-2",..: 4 4 5 5 5 6 7 8 9 10 ...
##  $ ID                     : Factor w/ 64 levels "020-01-01","020-01-02",..: 1 2 3 3 4 5 6 7 8 9 ...
##  $ GENERALIZED_ENVIRONMENT: Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
##  $ SENTINEL_PRESENCE      : Factor w/ 2 levels "NO","YES": 1 1 1 2 1 1 2 1 2 1 ...
##  $ BAIT_PRESENCE          : Factor w/ 2 levels "NO","YES": 1 1 1 1 1 2 2 2 2 2 ...
##  $ GROUP_SIZE             : Factor w/ 2 levels "LARGE","SMALL": 2 2 2 2 2 2 2 2 2 2 ...
##  $ DISTURBANCE_FREQUENCY  : num  0 0 0 0 0 ...
##  $ HD_BEHAVIOR_DURATION   : num  39.76 18.77 22.15 4.22 9.25 ...
##  $ PECK_RATE              : num  46.8 63.9 24.4 14.2 13 ...
```
Next, we'll compile the means of the peck rate by sentinel presence and generalized environment.


```
##   GENERALIZED_ENVIRONMENT  N PECK_RATE       sd       se       ci
## 1              Commercial 47  69.74557 23.75007 3.464303 6.973281
## 2              Green Area 32  53.43839 22.25837 3.934761 8.024998
```

## PECK Dot Plot


```r
PECK.DOTPLOT<-ggplot(data = PECK.SUMMARY
               , aes(x = GENERALIZED_ENVIRONMENT
                     , y = PECK_RATE
                     , color = GENERALIZED_ENVIRONMENT))+
  geom_point(position = position_dodge(width = 0.9)
             , size = 3) +
  geom_errorbar(aes(ymin=(PECK_RATE-se)
                    , ymax=(PECK_RATE+se))
                , width = 0.1
                , position = position_dodge(width=0.9))+
  geom_text(aes(label = round(PECK_RATE, 2)), hjust = -0.2, size = 6) +
  theme_classic() +
  ylab("Mean Peck Rate (per min)") +
  scale_colour_manual(values = cbPalette
                      , guide="none") +
  scale_x_discrete(labels = c("Commercial Area"
                                  , "Green Area")) +
  theme(axis.title.x = element_blank()
        , legend.position = "bottom"
        , legend.box="vertical"
        , legend.margin=margin()
        , text = element_text(size = 18))

PECK.DOTPLOT
```

![](GRaD-Presentation_files/figure-html/PECK Dot Plot-1.png)<!-- -->

## PECK Model


```r
PECK.MOD<-rlmer(PECK_RATE~GENERALIZED_ENVIRONMENT*DISTURBANCE_FREQUENCY+GROUP_SIZE+BAIT_PRESENCE+(1|VIDEO_ID/ID), data = PECK)

sjPlot::tab_model(PECK.MOD
                  , pred.labels = labs.peck
                  , show.re.var = T
                  , title = "Robust Peck Rate Model Output"
                  , dv.labels = " Effects on peck rate")
```

<table style="border-collapse:collapse; border:none;">
<caption style="font-weight: bold; text-align:left;">Robust Peck Rate Model Output</caption>
<tr>
<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">&nbsp;</th>
<th colspan="3" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; "> Effects on peck rate</th>
</tr>
<tr>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">Predictors</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">Estimates</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">CI</td>
<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">p</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Intercept</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">52.72</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">34.66&nbsp;&ndash;&nbsp;70.78</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>&lt;0.001</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Generalized Environment</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;12.96</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;27.28&nbsp;&ndash;&nbsp;1.36</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.076</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Disturbance Frequency</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">3.14</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;2.11&nbsp;&ndash;&nbsp;8.40</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.241</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Group Size</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;3.70</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">&#45;18.24&nbsp;&ndash;&nbsp;10.85</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">0.619</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Bait Presence</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">16.44</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">2.13&nbsp;&ndash;&nbsp;30.74</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.024</strong></td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">Generalized Environment:<br>Disturbance Frequency</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">16.31</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">4.35&nbsp;&ndash;&nbsp;28.27</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  "><strong>0.008</strong></td>
</tr>
<tr>
<td colspan="4" style="font-weight:bold; text-align:left; padding-top:.8em;">Random Effects</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&sigma;<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">266.50</td>
</tr>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>ID:VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.00</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">&tau;<sub>00</sub> <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">73.62</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">ICC</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.22</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">64</td>

<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">N <sub>VIDEO_ID</sub></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">25</td>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">Observations</td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="3">79</td>
</tr>
<tr>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">Marginal R<sup>2</sup> / Conditional R<sup>2</sup></td>
<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="3">0.341 / 0.484</td>
</tr>

</table>

```r
sjPlot::plot_model(PECK.MOD
                  , axis.labels = labs.peck
                  , show.values=T
                  , show.p=T
                  , value.offset = 0.4
                  , value.size = 3.5
                  , wrap.title = 48
                  , show.re.var = T
                  , title = "Effects on peck rate" )
```

![](GRaD-Presentation_files/figure-html/PECK Model-1.png)<!-- -->

## Disturbance Frequency x Generalized Environment


```r
PECK.DISTURBANCE.DOTPLOT<-PECK %>%
    ggplot(aes(x = DISTURBANCE_FREQUENCY
               , y = PECK_RATE
               , color = GENERALIZED_ENVIRONMENT))+
  geom_point(size = 3) +
  geom_smooth(method = "lm", se = F) +
  theme_classic() +
  labs(y="Peck Rate (per min)", x="Disturbance Frequency (per min)")+
  scale_x_continuous(n.breaks=14)+
  theme(legend.position = "bottom"
        , text = element_text(size = 18))+
  scale_color_manual(values = cbPalette
                    , labels = c("Commercial Area"
                                 , "Green Area")
                    , name="")
  

PECK.DISTURBANCE.DOTPLOT
```

```
## `geom_smooth()` using formula = 'y ~ x'
```

![](GRaD-Presentation_files/figure-html/PECK Disturbance plot-1.png)<!-- -->

# Sentinel Likelihood


```r
SENT<-DATA.SR[1:81,c(1,7,10,15,17,19,20,22,30,31)] %>%
  distinct()
str(SENT)
```

```
## 'data.frame':	33 obs. of  10 variables:
##  $ VIDEO_ID                       : Factor w/ 25 levels "037-2","038-2",..: 4 5 5 6 7 8 9 10 10 11 ...
##  $ DECIMAL_TIME                   : num  6.33 6.22 6.22 6.3 6.52 ...
##  $ TEMPERATURE                    : int  18 16 16 16 19 22 24 21 21 22 ...
##  $ GENERALIZED_ENVIRONMENT        : Factor w/ 2 levels "Commercial","Green Area": 2 2 2 2 2 2 2 2 2 2 ...
##  $ SENTINEL_PRESENCE              : Factor w/ 2 levels "NO","YES": 1 1 2 1 2 1 2 1 2 2 ...
##  $ BAIT_PRESENCE                  : Factor w/ 2 levels "NO","YES": 1 1 1 2 2 2 2 2 2 2 ...
##  $ NUMBER_OF_CROWS_RECORDED       : int  2 2 2 1 1 1 1 2 2 1 ...
##  $ GROUP_SIZE                     : Factor w/ 2 levels "LARGE","SMALL": 2 2 2 2 2 2 2 2 2 2 ...
##  $ TOTAL_FREQUENCY_OF_DISTURBANCES: num  0 0 0 0.142 0.223 ...
##  $ DISTURBANCE_FREQUENCY          : Factor w/ 3 levels "HIGH","LOW","MEDIUM": 2 2 2 3 3 3 3 3 3 2 ...
```


```r
SENT.SUMMARY <- summarySE(data = SENT
                      , measurevar = "GROUP_SIZE" #Random
                      , groupvars = c("GENERALIZED_ENVIRONMENT"
                                      , "SENTINEL_PRESENCE"
                                      )
                      )

SENT.SUMMARY
```

```
##   GENERALIZED_ENVIRONMENT SENTINEL_PRESENCE  N GROUP_SIZE        sd         se
## 1              Commercial                NO  6   1.666667 0.5163978 0.21081851
## 2              Commercial               YES  7   1.571429 0.5345225 0.20203051
## 3              Green Area                NO  8   2.000000 0.0000000 0.00000000
## 4              Green Area               YES 12   1.916667 0.2886751 0.08333333
##          ci
## 1 0.5419262
## 2 0.4943508
## 3 0.0000000
## 4 0.1834154
```

##SENT Barplot


```r
SENT.BARPLOT <- ggplot(SENT.SUMMARY, aes(x=GENERALIZED_ENVIRONMENT
                 , fill = SENTINEL_PRESENCE
                 , y = N
                 , group = SENTINEL_PRESENCE)) + 
  geom_bar(stat='identity', width=.5, position = "dodge") +
  geom_text(aes(label = N, group = SENTINEL_PRESENCE), position = position_dodge(width = 0.5), vjust = 1.5, size = 8) +
  ylab("Number of observations") +
  theme_classic() +
  scale_fill_manual(""
                    , values = cbPalette
                    , labels = c("Sentinel Absent", "Sentinel Present")
                      ) +
  theme(axis.title.x = element_blank()
        , text = element_text(size = 18)
        , legend.position = "bottom"
        , legend.box="vertical"
        , legend.margin=margin())
  
SENT.BARPLOT
```

![](GRaD-Presentation_files/figure-html/SENT Barplot-1.png)<!-- -->

##SENT Chi-Squared


```r
SENT.CHISQ<-lapply(SENT[,c("GENERALIZED_ENVIRONMENT"
                      , "GROUP_SIZE"
                      , "DISTURBANCE_FREQUENCY"
                      )
                              ]
                         , function(x) CrossTable(x,SENT$SENTINEL_PRESENCE
                                                  , fisher = T
                                                  , chisq = T
                                                  , expected = T
                                                  , prop.c = F
                                                  , prop.t = F
                                                  , prop.chisq = F
                                                  , sresid = T
                                                  , format = 'SPSS')
                         )
```

```
## Warning in chisq.test(tab, correct = FALSE, ...): Chi-squared approximation may
## be incorrect
```

```
## Warning in chisq.test(tab, correct = TRUE, ...): Chi-squared approximation may
## be incorrect
```

```
## Warning in chisq.test(tab, correct = FALSE, ...): Chi-squared approximation may
## be incorrect
```

```r
SENT.CHISQ
```

```
## $GENERALIZED_ENVIRONMENT
##    Cell Contents 
## |-------------------------|
## |                   Count | 
## |         Expected Values | 
## |             Row Percent | 
## |            Std Residual | 
## |-------------------------|
## 
## =====================================
##               SENT$SENTINEL_PRESENCE
## x                 NO      YES   Total
## -------------------------------------
## Commercial        6        7      13 
##                 5.5      7.5         
##                46.2%    53.8%   39.4%
##               0.206   -0.177         
## -------------------------------------
## Green Area        8       12      20 
##                 8.5     11.5         
##                40.0%    60.0%   60.6%
##              -0.166    0.143         
## -------------------------------------
## Total            14       19      33 
## =====================================
## 
## Statistics for All Table Factors
## 
## Pearson's Chi-squared test 
## ------------------------------------------------------------
## Chi^2 = 0.1221515      d.f. = 1      p = 0.727 
## 
## Pearson's Chi-squared test with Yates' continuity correction 
## ------------------------------------------------------------
## Chi^2 = 0      d.f. = 1      p = 1 
## 
##  
## Fisher's Exact Test for Count Data
## ------------------------------------------------------------
## Sample estimate odds ratio: 1.275883 
## 
## Alternative hypothesis: true odds ratio is not equal to 1 
## p = 1 
## 95% confidence interval: 0.2492462 6.528091 
## 
## Alternative hypothesis: true odds ratio is less than 1 
## p = 0.761 
## 95%s confidence interval: % 0 5.198388 
## 
## Alternative hypothesis: true odds ratio is greater than 1 
## p = 0.503 
## 95%s confidence interval: % 0.3131908 Inf 
## 
##         Minimum expected frequency: 5.515152 
## 
## 
## $GROUP_SIZE
##    Cell Contents 
## |-------------------------|
## |                   Count | 
## |         Expected Values | 
## |             Row Percent | 
## |            Std Residual | 
## |-------------------------|
## 
## ================================
##          SENT$SENTINEL_PRESENCE
## x            NO      YES   Total
## --------------------------------
## LARGE        2        4       6 
##            2.5      3.5         
##           33.3%    66.7%   18.2%
##         -0.342    0.293         
## --------------------------------
## SMALL       12       15      27 
##           11.5     15.5         
##           44.4%    55.6%   81.8%
##          0.161   -0.138         
## --------------------------------
## Total       14       19      33 
## ================================
## 
## Statistics for All Table Factors
## 
## Pearson's Chi-squared test 
## ------------------------------------------------------------
## Chi^2 = 0.2481203      d.f. = 1      p = 0.618 
## 
## Pearson's Chi-squared test with Yates' continuity correction 
## ------------------------------------------------------------
## Chi^2 = 0.001723058      d.f. = 1      p = 0.967 
## 
##  
## Fisher's Exact Test for Count Data
## ------------------------------------------------------------
## Sample estimate odds ratio: 0.6337315 
## 
## Alternative hypothesis: true odds ratio is not equal to 1 
## p = 1 
## 95% confidence interval: 0.04933611 5.337857 
## 
## Alternative hypothesis: true odds ratio is less than 1 
## p = 0.49 
## 95%s confidence interval: % 0 4.034101 
## 
## Alternative hypothesis: true odds ratio is greater than 1 
## p = 0.829 
## 95%s confidence interval: % 0.07362799 Inf 
## 
##         Minimum expected frequency: 2.545455 
## Cells with Expected Frequency < 5: 2 of 4 (50%)
## 
## 
## $DISTURBANCE_FREQUENCY
##    Cell Contents 
## |-------------------------|
## |                   Count | 
## |         Expected Values | 
## |             Row Percent | 
## |            Std Residual | 
## |-------------------------|
## 
## =================================
##           SENT$SENTINEL_PRESENCE
## x             NO      YES   Total
## ---------------------------------
## HIGH          1        5       6 
##             2.5      3.5         
##            16.7%    83.3%   18.2%
##          -0.969    0.831         
## ---------------------------------
## LOW           7        7      14 
##             5.9      8.1         
##            50.0%    50.0%   42.4%
##           0.435   -0.374         
## ---------------------------------
## MEDIUM        6        7      13 
##             5.5      7.5         
##            46.2%    53.8%   39.4%
##           0.206   -0.177         
## ---------------------------------
## Total        14       19      33 
## =================================
## 
## Statistics for All Table Factors
## 
## Pearson's Chi-squared test 
## ------------------------------------------------------------
## Chi^2 = 2.032678      d.f. = 2      p = 0.362 
## 
## 
##  
## Fisher's Exact Test for Count Data
## ------------------------------------------------------------
## Alternative hypothesis: two.sided 
## p = 0.476 
##         Minimum expected frequency: 2.545455 
## Cells with Expected Frequency < 5: 2 of 6 (33.33333%)
```

##SENT Plots - Continuous Variables


```r
SENT.FREQ <- SENT %>% 
  rename("Temperature (deg. C)" = "TEMPERATURE"
         , "Decimal Time" = "DECIMAL_TIME"
         , "Disturbance Frequency (per min)" = "TOTAL_FREQUENCY_OF_DISTURBANCES"
         , "Number of Crows" = "NUMBER_OF_CROWS_RECORDED") %>%
  pivot_longer(., cols = c(#"Temperature (deg. C)"
                           # , "Decimal Time"
                           "Number of Crows"
                           , "Disturbance Frequency (per min)")
               , names_to = "Var"
               , values_to = "Val") %>%
  ggplot(aes(
    x = Val
    , colour = SENTINEL_PRESENCE))+
  geom_freqpoly(linewidth = 1
                , alpha=I(.6)) +
  scale_colour_manual(values = cbPalette
                      , name = ""
                      , labels = c("Sentinel Absent", "Sentinel Present")) +
  facet_wrap(~Var, scale = "free_x") +
  theme_classic() +
  theme(legend.position = "bottom"
        , axis.title.x = element_blank()
        , text = element_text(size = 18))

SENT.FREQ
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](GRaD-Presentation_files/figure-html/SENT Frequency plots-1.png)<!-- -->
