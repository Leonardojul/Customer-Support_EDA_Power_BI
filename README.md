# Portfolio
## Compilation of works and projects on data analysis

### Background

The support department has been getting complaints from customers on ticket handling speed and budget overrun. Both the project and support department share resources (developers) which never seem to be enough to fulfill the required allocated work. Management does not give credit to customer complaints for these reasons and argues that there are enough resources allocated to perform the upcoming (forecasted) work.

In this scenario, the first posed question is whether the workload forecasted by the developers is actually accurate. Knowing how reliable the time estimations are can help better predict the following weeks’ workload and, as a consequence, to make a better allocation of resources, allowing the company to fulfill promises made to customers on completion time and/or budget.



The first metric is actually the [M.A.P.E](https://en.wikipedia.org/wiki/Mean_absolute_percentage_error): 

![](https://latex.codecogs.com/png.image?\dpi{110}&space;\bg_white&space;M.A.P.E&space;=&space;\frac{1}{n}\sum_{t&space;=&space;1}^{n}\left|1-\frac{Ft}{At}&space;\right|)
 
The absolute part calculation has been implemented on Power BI on DAX on column level with the following code:

```DAX
Accuracy variation =
ABS (
    1
        - DIVIDE (
            RELATED ( vwHoursBookedPerTicket[SumBillableHours] ),
            Tickets[Total Estimation],
            0
        )
)
```

Then, an average function can be called on different filtered tables to answer any business question:

```DAX
Accuracy variation =
AVERAGE ( Tickets[Accuracy variation] )
```

Different visualizations were then built to illustrate the magnitude of the issue attending to different criteria.

Globally, per development team and per each support manager:

![Dashboard 1](https://github.com/Leonardojul/portfolio/blob/main/Accuracy-variation-1.png)




Detailed view (on drill down) for any of the given categories, breaking down the missed forecasted estimations on overestimates and underestimates:






Additionally to the Accuracy Variation KPI, a dashboard containing information on how much of the work has actually been estimated was necessary to understand how much the Accuracy Variation correlates with the actual situation on support.




The above data shows the magnitude of the situation in support, where 694.35 hours were estimated while over 2k hours were actually worked. The above means that the real work performed by support and the developers is around x3 times higher than what has been estimated.
