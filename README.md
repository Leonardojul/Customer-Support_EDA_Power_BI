# Portfolio
## Compilation of works and projects on data analysis

### Background

The support department has been getting complaints from customers on ticket handling speed and budget overrun. Both the project and support department share resources (developers) which never seem to be enough to fulfill the required allocated work. Management does not give credit to customer complaints for these reasons and argues that there are enough resources allocated to perform the upcoming (forecasted) work.

In this scenario, the first posed question is whether the workload forecasted by the developers is actually accurate. Knowing how reliable the time estimations are can help better predict the following weeks’ workload and, as a consequence, to make a better allocation of resources, allowing the company to fulfill promises made to customers on completion time and/or budget.

**INDEX**
1. [Accuracy variation](#accuracy-variation)
2. [Ticket bottleneck analysis](#ticket-bottleneck-analysis)
3. [Ticket categorization](#ticket-categorization)
4. [Conclussions and recommendations](#conclussions-and-recommendations)


### Accuracy variation

The first metric chosen to analyze the scenario is the [M.A.P.E](https://en.wikipedia.org/wiki/Mean_absolute_percentage_error): 

![](https://latex.codecogs.com/png.image?\dpi{110}&space;\bg_white&space;M.A.P.E&space;=&space;\frac{1}{n}\sum_{t&space;=&space;1}^{n}\left&#124;1-\frac{Ft}{At}&space;\right&#124;)
 
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

![Dashboard 1](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Accuracy-variation-1.png)

Detailed view (on drill down) for any of the given categories, breaking down the missed forecasted estimations on overestimates and underestimates:

![Dashboard 2](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Accuracy-variation-2.png)

Additionally to the Accuracy Variation KPI, a dashboard containing information on how much of the work has actually been estimated was necessary to understand how much the Accuracy Variation correlates with the actual situation on support.

![Dashboard 3](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Work-completion.png)

The above data shows the magnitude of the situation in support, where 739 hours were estimated while over 2k hours were actually worked (1.3k + 718.7 hours). This means that the real work performed by support and the developers is around x3 times higher than what has been estimated.


### Ticket bottleneck analysis

The second request posed by management was to understand where the incoming work was getting stuck and what could be done to clear possible bottlenecks, if any.

To follow up a ticket's activity in real time, a new column that counts running stops (one stop is counted every time a ticket has to wait in a queie, ie; developer support, hosting, support managers...) had to be implemented:

```DAX
AccumulatedStops =
VAR Ticket = TicketHistory[Work Item Id]
VAR CurrentDate = TicketHistory[Date]
VAR FilteredTable =
    FILTER (
        TicketHistory,
        TicketHistory[Work Item Id] = Ticket
            && TicketHistory[Date] <= CurrentDate
    )
RETURN
    CALCULATE ( SUM ( TicketHistory[Change column] ), FilteredTable )
```

Then, the current number of stops needs to be updated in the tickets table:

```DAX
CurrentStops =
CALCULATE (
    MAX ( TicketHistory[AccumulatedStops] ),
    FILTER ( TicketHistory, TicketHistory[Work Item Id] = Tickets[Work Item Id] )
)
```

Finally, it is possible to count how many tickets are there per each number of stops, e.g. How many tickets required three stops to be resolved?

```DAX
Count =
CALCULATE (
    COUNT ( Tickets[Work Item Id] ),
    FILTER ( Tickets, Tickets[CurrentStops] = CountofTicketStops[Number of stops] )
)
```
When plotting the data in a histogram, the following distribution was revealed:

![Dashboard 4](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Ticket-history-analysis-1.png)

- On average, a ticket has to wait up to 4 times to be picked up. 
- Most tickets will be resolved after queuing in 2 stops
- Tickets resolved after the highest number of stops are requests for change (RFC in short)

Complementing the above, it was necessary to know where these tickets were actually waiting, in other words, which queues seemed to be the slowest, blocking their progress on resolution. For this it was necessary to analyze the distributions of waiting times depending on queues.

It was necessary tp categorize the queues or buckets and to classify each stop for each ticket accordingly:

```DAX
Queue =
IF (
    TicketHistory[SPP SC priority] <> BLANK (),
    "Dev team",
    SWITCH (
        TicketHistory[Assigned To],
        "hosting", "Hosting",
        BLANK (), "Service Consultant",
        "Customer", "Customer",
        "AX-FO-Support", "ERP Support",
        "B1-Support", "ERP Support",
        "ECC-Support", "ERP Support",
        "GP-Support", "ERP Support",
        "NAV-BC-Support", "ERP Support",
        "Add-on support", "Add-On Support",
        "Core support", "Core support",
        IF (
            CONTAINS ( Tickets, Tickets[ServiceManager], TicketHistory[Assigned To] ),
            TicketHistory[Assigned To]
        ), "Service Consultant",
        "Other"
    )
)
```

The next step is to aggregate the data and to plot it:

![Dashboard 5](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Ticket-history-analysis-2.png)


### Ticket categorization

Another question presented by the business was to better understand the work performed on support. Since this information was not readily available, customization were performed on the Azure DevOps platform to gather the missing data and a new project started.

Every support manager was requested to fill in two new fields added at ticket level to help categorize and describe the work being done on support.

After a few weeks, the first descriptive metrics could be presented:

![Dashboard 6](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Categorization-1.png)

Drilling down on the data:

![Dashboard 7](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Categorization-2.png)

And finally, an overall description of the most frequent issues being handled in support:

![Dashboard 8](https://raw.githubusercontent.com/Leonardojul/portfolio/main/Categorization-3.png)


### Conclussions and recommendations

- Accuracy on time estimations needs to be improved in order to be able to properly forecast resources needed on support, ticket budget or completion time
- For the same reasons the estimated ticket ratio also needs to increase.
- The greatest bottleneck to ticket completion is the waiting time on the customer's side. Customer success management could raise awareness on this fact to speed ticket handling on support and thus increase customer satisfaction
- The development team is the second greatest bottleneck. More resources should be assigend to support to reduce ticket handling time
- The majority of the work on support is on issues reported by customers. Knowing which issues are more commmon, take more resources or take longer to get resolved should help the business focus on the problematic areas for some quick wins.

A set of new KPIs has been proposed to help track resources efficiency and workload prediction:
- Accuracy variation
- Estimated ticket ratio
- Number of stops per ticket
