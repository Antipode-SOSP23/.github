## Antipode



You can read and our paper [Antipode: Enforcing Cross-Service Causal Consistency
in Distributed Applications](PDF).
```BibTeX
@inproceedings{loff2023antipode,
}
```


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

For the full results obtained in the paper, execute the combinations using the available post and notification storages -- with and without Antipode (`-ant`) enabled:

| Post-Storage | Notification-Storage |
| :----------: | :------------------: |
| mysql        | sns                  |
| dynamo       | mq                   |
| s3           | dynamo               |
| cache        |                      |

For the results of percentage of inconsistencies obtained in the paper, we used the following storage combinations and delays:
- cache-sns: 100 -> 1500 (increments of 100)
- dynamo-sns: 100, 200, 300, 400, 500 -> 3000 (increments of 250)
- mysql-sns: 100 -> 1500 (increments of 100)
- s3-sns: 500, 1000, 10k -> 50k (increments of 5k)

**Note**: The `maestrina` is a convenience script to run all combinations using a single script. 
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
