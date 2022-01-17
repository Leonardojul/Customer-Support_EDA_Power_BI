# Portfolio
## Compilation of works and projects on data analysis

### Background

The support department has been getting complaints from customers on ticket handling speed and budget overrun. Both the project and support department share resources (developers) which never seem to be enough to fulfill the required allocated work. Management does not give credit to customer complaints for these reasons and argues that there are enough resources allocated to perform the upcoming (forecasted) work.

In this scenario, the first posed question is whether the workload forecasted by the developers is actually accurate. Knowing how reliable the time estimations are can help better predict the following weeks’ workload and, as a consequence, to make a better allocation of resources, allowing the company to fulfill promises made to customers on completion time and/or budget.

**INDEX**
1. [Accuracy variation](#accuracy-variation)
2. [Ticket bottleneck analysis](#ticket-bottleneck-analysis)
3. Ticket categorization
4. Conclussions and recommendations


### Accuracy variation

The first metric chosen to analyze the scenario is the [M.A.P.E](https://en.wikipedia.org/wiki/Mean_absolute_percentage_error): 

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

![Dashboard 2](https://github.com/Leonardojul/portfolio/blob/main/Accuracy-variation-2.png)

Additionally to the Accuracy Variation KPI, a dashboard containing information on how much of the work has actually been estimated was necessary to understand how much the Accuracy Variation correlates with the actual situation on support.

![Dashboard 3](https://github.com/Leonardojul/portfolio/blob/main/Work-completion.png)

The above data shows the magnitude of the situation in support, where 739 hours were estimated while over 2k hours were actually worked (1.3k + 718.7 hours). This means that the real work performed by support and the developers is around x3 times higher than what has been estimated.


### Ticket bottleneck analysis

The second request posed by management was to understand where the incoming work was getting stuck and what could be done to clear possible bottlenecks, if any.

To follow up a ticket's activity in real time, a new column that counts running stops (one stop is counted every time a ticket has to wait in a queie, ie; developer support, hosting, support managers...) had to be implemented:

[Insert dax code here]

When plotting the data in a histogram, the following distribution was revealed:

![Dashboard 4](https://github.com/Leonardojul/portfolio/blob/main/Ticket-history-analysis-1.png)

- On average, a ticket has to wait up to 4 times to be picked up. 
- Most tickets will be resolved after queuing in 2 stops
- Tickets resolved after the highest number of stops are requests for change (RFC in short)

Complementing the above, it was necessary to know where these tickets were actually waiting, in other words, which queues seemed to be the slowest, blocking their progress on resolution. For this it was necessary to analyze the distributions of waiting times depending on queues.

The first step was to categorize the queues or buckets and to classify each stop for each ticket accordingly:

[insert dax code here]

The next step is to aggregate the information

[dax code]

And finally it is possible to visualize it:

![Dashboard 4](https://github.com/Leonardojul/portfolio/blob/main/Ticket-history-analysis-2.png)

