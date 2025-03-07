# Measures Documentation

This document provides detailed information about all measures used in the Sales KPIs dashboard.

## Table of Contents
- [Goal Measures](#goal-measures)
- [EaaS Measures](#eaas-measures)
- [PPA Measures](#ppa-measures)
- [Carbon Credit Measures](#carbon-credit-measures)
- [Visual Indicator Measures](#visual-indicator-measures)

## Goal Measures

### EaaS Goal
```dax
VAR SelectedVerticals = ALLSELECTED(Account[Vertical])
VAR RE_goal = 4800000
VAR COMM_goal = 43021231
VAR AM_goal = 22900000
VAR Other_goal = 4100000
VAR Total = RE_goal + COMM_goal + AM_goal + Other_goal
VAR IsAnyFilterApplied = COUNTROWS(ALLSELECTED(Account[Vertical])) < COUNTROWS(ALL(Account[Vertical]))
VAR isEcommerceSelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "E-Commerce")
VAR isRenewableSelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Renewable Energy")
VAR IsAutomotiveMachinerySelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Automotive Machinery")
VAR IsOtherSelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Other")
RETURN
SWITCH(
    TRUE(),
    isEcommerceSelected, COMM_goal,
    isRenewableSelected, RE_goal,
    IsAutomotiveMachinerySelected, AM_goal,
    IsOtherSelected, Other_goal,
    Total
)
```
This measure calculates the EaaS (Energy as a Service) goal based on the selected vertical market. Each vertical has a predefined goal amount, and the measure returns the appropriate goal value based on the filter context. If no specific vertical is selected, it returns the sum of all goals.

### PPA Goal
```dax
VAR SelectedVerticals = ALLSELECTED(Account[Vertical])
VAR RE_goal  = 956341
VAR COMM_goal  = 1424116
VAR AM_goal = 1985447
VAR Other_goal = 634096
VAR NoCategory = 0
VAR Total = RE_goal  + COMM_goal  + AM_goal + Other_goal
VAR IsAnyFilterApplied = COUNTROWS(ALLSELECTED(Account[Vertical])) < COUNTROWS(ALL(Account[Vertical]))
VAR isEcommerceSelected  = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "E-Commerce")
VAR isRenewableSelected  = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Renewable Energy")
VAR IsAutomotiveMachinerySelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Automotive Machinery")
VAR IsOtherSelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Other")
VAR IsNoCategorySelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "NoCategory")
RETURN
SWITCH(
    TRUE(),
    isEcommerceSelected, COMM_goal,
    isRenewableSelected, RE_goal,
    IsAutomotiveMachinerySelected, AM_goal,
    IsOtherSelected, Other_goal,
    IsNoCategorySelected, NoCategory,
    Total
)
```
This measure calculates the PPA (Power Purchase Agreement) goal based on the selected vertical market. Each vertical has a predefined goal amount, and the measure returns the appropriate goal value based on the filter context. If no specific vertical is selected, it returns the sum of all goals.

### Carbon Credit Goal
```dax
VAR SelectedVerticals = ALLSELECTED(Account[Vertical])
VAR RE_goal = 321966
VAR COMM_goal  = 412371
VAR AM_goal = 0
VAR Other_goal = 477663
VAR Total = RE_goal  + COMM_goal  + AM_goal + Other_goal
VAR IsAnyFilterApplied = COUNTROWS(ALLSELECTED(Account[Vertical])) < COUNTROWS(ALL(Account[Vertical]))
VAR isEcommerceSelected  = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "E-Commerce")
VAR isRenewableSelected   = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Renewable Energy")
VAR IsAutomotiveMachinerySelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Automotive Machinery")
VAR IsOtherSelected = IsAnyFilterApplied && CONTAINS(SelectedVerticals, Account[Vertical], "Other")
RETURN
SWITCH(
    TRUE(),
    isEcommerceSelected, COMM_goal,
    isRenewableSelected, RE_goal,
    IsAutomotiveMachinerySelected, AM_goal,
    IsOtherSelected, Other_goal,
    Total
)
```
This measure calculates the Carbon Credit goal based on the selected vertical market. Each vertical has a predefined goal amount, and the measure returns the appropriate goal value based on the filter context. Note that Automotive Machinery has a goal of 0 for Carbon Credits. If no specific vertical is selected, it returns the sum of all goals.

## EaaS Measures

### Won Value EaaS Services
```dax
VAR Consultancy=
CALCULATE(
    SUM('OpportunityRevenue'[Total Sum]), 
    'OpportunityRevenue'[Revenue Type]= "EaaS Services", 
    Opportunity[Status]="Won"
    )
RETURN
IF(ISBLANK(Consultancy), 0, Consultancy)
```
This measure calculates the total value of won opportunities for EaaS Services. It sums the Total Sum column from the OpportunityRevenue table where the Revenue Type is "EaaS Services" and the Opportunity Status is "Won". If no matching records are found, it returns 0.

### Weighted Opportunity Pipe EaaS
```dax
VAR TotalAmount100 = 
    CALCULATE(
    SUM(
        'OpportunityRevenue'[Total Sum]
    ), 
        Opportunity[Status] = "In Progress",
        'OpportunityRevenue'[Revenue Type]= "EaaS Services"
        )

VAR TotalAmountWeighted =
    CALCULATE(
    SUMX(
        'OpportunityRevenue',
        'OpportunityRevenue'[Total Sum]*RELATED(Opportunity[Probability])
    ), 
        Opportunity[Status] = "In Progress",
        'OpportunityRevenue'[Revenue Type]= "EaaS Services"
        )

RETURN
IF(
    ISFILTERED(Toggle[Opportunity]),
    TotalAmount100,
    TotalAmountWeighted
)
```
This measure calculates the weighted pipeline value for EaaS Services. It provides two calculations:
1. TotalAmount100: The unweighted sum of all in-progress opportunities for EaaS Services
2. TotalAmountWeighted: The weighted sum where each opportunity's value is multiplied by its probability

The measure returns either the weighted or unweighted value based on the Toggle selection.

### Remaining Amount EaaS
```dax
VAR RemainingTotal= [EaaS Goal] - [Won Value EaaS Services] - [Weighted Opportunity Pipe EaaS]
RETURN
IF(RemainingTotal < 0,
    0,
    RemainingTotal
)
```
This measure calculates the remaining amount needed to reach the EaaS goal. It subtracts the won value and the weighted pipeline value from the goal. If the result is negative (goal exceeded), it returns 0.

### 100% Pipe EaaS
```dax
IF(
    [Weighted Opportunity Pipe EaaS] <= -[Goal - Won EaaS], [Weighted Opportunity Pipe EaaS], 
    -[Goal - Won EaaS])
```
This measure calculates the portion of the EaaS pipeline that contributes to reaching the goal. It returns either the full weighted pipeline value or just the amount needed to reach the goal, whichever is smaller.

## PPA Measures

### Won Revenue Value Per Year PPA
```dax
CALCULATE(
    SUMX('OpportunityRevenue', 
        'OpportunityRevenue'[Total Sum]
        /
        'OpportunityRevenue'[Total Revenue Years]),
    'OpportunityRevenue'[Revenue Type] = "Carbon Credit",
    Opportunity[Status] = "Won",
    'OpportunityRevenue'[Carbon Credit] = "No"
)
```
This measure calculates the annual value of won PPA opportunities. It divides the total sum by the number of revenue years to get an annual value, then sums these values for all won opportunities where Carbon Credit is "No".

### Weighted OpportunityRevenue Value Per Year PPA
```dax
VAR TotalAmount100 = 
    CALCULATE(
            SUMX('OpportunityRevenue',
            'OpportunityRevenue'[Total Sum]/'OpportunityRevenue'[Total Revenue Years]
            ),
            'OpportunityRevenue'[Revenue Type] = "PPA",
            Opportunity[Status] = "In Progress",
            'OpportunityRevenue'[Carbon Credit]="No"
    )

VAR TotalAmountWeighted = 
    CALCULATE(
        SUMX(
            'OpportunityRevenue', 
            ('OpportunityRevenue'[Total Sum] / 'OpportunityRevenue'[Total Revenue Years]) * RELATED(Opportunity[Probability])
        ),
        'OpportunityRevenue'[Revenue Type] = "PPA",
        Opportunity[Status] = "In Progress",
        'OpportunityRevenue'[Carbon Credit]="No"
    )

RETURN
IF(
    ISFILTERED(Toggle[Opportunity]),
    TotalAmount100,
    TotalAmountWeighted
)
```
This measure calculates the weighted annual pipeline value for PPA opportunities. It provides two calculations:
1. TotalAmount100: The unweighted annual sum of all in-progress PPA opportunities
2. TotalAmountWeighted: The weighted annual sum where each opportunity's annual value is multiplied by its probability

The measure returns either the weighted or unweighted value based on the Toggle selection.

### Remaining Amount PPA
```dax
VAR RemainingTotal = [PPA Goal] - [Won Revenue Value Per Year PPA] - [Weighted OpportunityRevenue Value Per Year PPA]
RETURN
IF(RemainingTotal < 0,
    0,
    RemainingTotal
)
```
This measure calculates the remaining amount needed to reach the PPA goal. It subtracts the won value and the weighted pipeline value from the goal. If the result is negative (goal exceeded), it returns 0.

### 100% Pipe PPA
```dax
IF(
    [Weighted OpportunityRevenue Value Per Year PPA] <= -[Goal - Won PPA], [Weighted OpportunityRevenue Value Per Year PPA], 
    -[Goal - Won PPA])
```
This measure calculates the portion of the PPA pipeline that contributes to reaching the goal. It returns either the full weighted pipeline value or just the amount needed to reach the goal, whichever is smaller.

## Carbon Credit Measures

### Won Revenue Value Per Year
```dax
CALCULATE(
    SUMX('OpportunityRevenue', 
        'OpportunityRevenue'[Total Sum]
        /
        'OpportunityRevenue'[Total Revenue Years]),
        'OpportunityRevenue'[Revenue Type] = "Carbon Credit",
        Opportunity[Status] = "Won",
        'OpportunityRevenue'[Carbon Credit] = "Yes"
    )
```
This measure calculates the annual value of won Carbon Credit opportunities. It divides the total sum by the number of revenue years to get an annual value, then sums these values for all won opportunities where Carbon Credit is "Yes".

### Weighted OpportunityRevenue Value Per Year
```dax
VAR TotalAmount100 = 
    CALCULATE(
            SUMX('OpportunityRevenue',
            'OpportunityRevenue'[Total Sum]/'OpportunityRevenue'[Total Revenue Years]
            ),
            'OpportunityRevenue'[Revenue Type] = "Carbon Credit",
            Opportunity[Status] = "In Progress",
            'OpportunityRevenue'[Carbon Credit]="Yes"
    )

VAR TotalAmountWeighted = 
    CALCULATE(
        SUMX(
            'OpportunityRevenue', 
            ('OpportunityRevenue'[Total Sum] / 'OpportunityRevenue'[Total Revenue Years]) * RELATED(Opportunity[Probability])
        ),
        'OpportunityRevenue'[Revenue Type] = "Carbon Credit",
        Opportunity[Status] = "In Progress",
        'OpportunityRevenue'[Carbon Credit]="Yes"
    )

RETURN
IF(
    ISFILTERED(Toggle[Opportunity]),
    TotalAmount100,
    TotalAmountWeighted
)
```
This measure calculates the weighted annual pipeline value for Carbon Credit opportunities. It provides two calculations:
1. TotalAmount100: The unweighted annual sum of all in-progress Carbon Credit opportunities
2. TotalAmountWeighted: The weighted annual sum where each opportunity's annual value is multiplied by its probability

The measure returns either the weighted or unweighted value based on the Toggle selection.

### Remaining Amount Carbon Credit
```dax
VAR RemainingTotal = [Carbon Credit Goal] - [Won Revenue Value Per Year] - [Weighted OpportunityRevenue Value Per Year]
RETURN
IF(RemainingTotal < 0,
    0,
    RemainingTotal
)
```
This measure calculates the remaining amount needed to reach the Carbon Credit goal. It subtracts the won value and the weighted pipeline value from the goal. If the result is negative (goal exceeded), it returns 0.

### 100% Pipe Carbon Credit
```dax
IF(
    [Weighted OpportunityRevenue Value Per Year] <= -[Goal - Won Carbon Credit], 
    [Weighted OpportunityRevenue Value Per Year], 
    -[Goal - Won Carbon Credit])
```
This measure calculates the portion of the Carbon Credit pipeline that contributes to reaching the goal. It returns either the full weighted pipeline value or just the amount needed to reach the goal, whichever is smaller.

## Visual Indicator Measures

### Goal - Won EaaS
```dax
VAR Remaining = [EaaS Goal] - [Won Value EaaS Services]
RETURN
IF(Remaining>=0, -Remaining, 0)
```
This measure calculates how much of the EaaS goal has been achieved. If the goal has been exceeded, it returns 0. Otherwise, it returns the negative of the remaining amount, which is used for visualization purposes.

### Goal - Won PPA
```dax
VAR Remaining = [PPA Goal] - [Won Revenue Value Per Year PPA]
RETURN
IF(Remaining>=0, -Remaining, 0)
```
This measure calculates how much of the PPA goal has been achieved. If the goal has been exceeded, it returns 0. Otherwise, it returns the negative of the remaining amount, which is used for visualization purposes.

### Goal - Won Carbon Credit
```dax
VAR Remaining = [Carbon Credit Goal] - [Won Revenue Value Per Year]
RETURN
IF(Remaining>=0, -Remaining, 0)
```
This measure calculates how much of the Carbon Credit goal has been achieved. If the goal has been exceeded, it returns 0. Otherwise, it returns the negative of the remaining amount, which is used for visualization purposes.

### Goal Checkmark EaaS
```dax
IF([Goal - Won EaaS] >=0, "<span style=""font-size: 70px; line-height: 70px; color: #C1CCE3;"">&#10003;</span>","")
```
This measure displays a checkmark symbol when the EaaS goal has been achieved.

### Goal Checkmark PPA
```dax
IF([Goal - Won PPA] >=0, "<span style=""font-size: 70px; line-height: 70px; color: #C1CCE3;"">&#10003;</span>","")
```
This measure displays a checkmark symbol when the PPA goal has been achieved.

### Goal Checkmark Carbon Credit
```dax
SWITCH(
    TRUE(),
    SELECTEDVALUE(Account[Vertical]) = "Automotive Machinery", "No Goal",  // If the vertical is Automotive Machinery, no goal is displayed as there is none for AM.
    [Goal - Won Carbon Credit] >= 0, "<span style='font-size: 70px; line-height: 70px; color: #C1CCE3;'>&#10003;</span>",
    ""
)
```
This measure displays a checkmark symbol when the Carbon Credit goal has been achieved. For the Automotive Machinery vertical, it displays "No Goal" since there is no Carbon Credit goal for this vertical.

### Sum of Product Goals 2024
```dax
SUM('Product'[Product Goals 2024 Carbon Credit])
```
This measure sums up all the product-specific Carbon Credit goals for 2024. 