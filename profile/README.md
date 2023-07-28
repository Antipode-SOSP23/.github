## Antipode


## SOSP 23 

Here is a quick rundown on how you can obtain the main results from the SOSP23 paper. 
There are currently 3 main results that come from 3 different repos:
- [Antipode @ Post-Notification](https://github.com/Antipode-SOSP23/antipode-post-notification)
- [Antipode @ DeathStarBench](https://github.com/Antipode-SOSP23/antipode-deathstarbench)
- [Antipode @ TrainTicket](https://github.com/Antipode-SOSP23/antipode-train-ticket)
There is also a minor result related to the [Alibaba microservice dataset](https://github.com/alibaba/clusterdata/tree/master/cluster-trace-microservices-v2021) which is used as motivation, which you can obtain here:
- [Antipode @ TrainTicket](https://github.com/Antipode-SOSP23/alibaba-spike)


#### Antipode @ Post-Notification

You should checkout the [`sosp23` release](https://github.com/Antipode-SOSP23/antipode-post-notification/tree/sosp23) and follow the instructions in that repo.
Next we provide a quick usage reference to get the main results from the paper. **Don't forget to check the pre-requisites and AWS configurations mentioned in instructions.**

```zsh
# Build deployment image -- one time only 
docker build -t antipode-lambda .

# Run maestrina for all combinations -- if you find errors use the antipode_lambda below
./maestrina

# For a single combination
./antipode_lambda build --post-storage mysql --notification-storage sns --writer eu --reader us
./antipode_lambda run -r 1000
./antipode_lambda gather
```
**Note**: The `maestrina` is a convenience script to run all experiments using a single script. 
If you find any errors we recommend you to use the `antipode_lambda` script as described in the instructions.

After gathering all the results duplicate the `plots/configs/sample.yml` file:
- Keep the `delay_vs_per_inconsistencies` as is
- Change the gather paths in `consistency_window` to the ones returned by the `gather` command by selecting only the results where the notification storage is `SNS`. In the end you will have a configuration similar to this:
```yml
  gather_paths:
    - 'cache-sns/eu-us__antipode__1000-20210913103954'
    - 'cache-sns/eu-us__1000-20210912134647'
    - 'dynamo-sns/eu-us__antipode__1000-20210913103954'
    - 'dynamo-sns/eu-us__1000-20210912134647'
    - 'mysql-sns/eu-us__antipode__1000-20210913103954'
    - 'mysql-sns/eu-us__1000-20210912134647'
    - 's3-sns/eu-us__antipode__1000-20210913103954'
    - 's3-sns/eu-us__1000-20210912134647'
```

Now you just plot your results with:
```zsh
./plot plots/configs/sample.yml --plots delay_vs_per_inconsistencies
./plot plots/configs/sample.yml --plots consistency_window
```
