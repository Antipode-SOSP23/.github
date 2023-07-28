## Antipode


Antipode prevents **cross-service inconsistencies** in distributed applications by enforcing cross-service consistency. It propagates metadata both alongside end-to-end requests and within datastores. Antipode enables a novel **cross-service causal consistency** model, which extends existing causality models, and whose enforcement requires us to bring in a series of contributions to address fundamental semantic, scalability, and deployment challenges.

You can read our paper [Antipode: Enforcing Cross-Service Causal Consistency
in Distributed Applications](PDF) which was published at SOSP'23.

And you can quickly add it to your references:
```BibTeX
@inproceedings{loff2023antipode,
}
```


## SOSP 23 

**NOTE for AE:** In the shepherding process we were requested an additional plot. We are in the development process of that.

Here is a quick rundown on how you can obtain the main results from the SOSP23 paper. 
There are currently 3 main results that come from 3 different repos:
- [Antipode @ Post-Notification](https://github.com/Antipode-SOSP23/antipode-post-notification)
- [Antipode @ DeathStarBench](https://github.com/Antipode-SOSP23/antipode-deathstarbench)
- [Antipode @ TrainTicket](https://github.com/Antipode-SOSP23/antipode-train-ticket)

There is also a minor result related to the [Alibaba microservice dataset](https://github.com/alibaba/clusterdata/tree/master/cluster-trace-microservices-v2021) which is used as motivation, which you can obtain here:
- [Antipode @ TrainTicket](https://github.com/Antipode-SOSP23/alibaba-spike)


### Antipode @ Post-Notification

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
- cache-sns: 100 &rarr; 1500 (increments of 100)
- dynamo-sns: 100, 200, 300, 400, 500 &rarr; 3000 (increments of 250)
- mysql-sns: 100 &rarr; 1500 (increments of 100)
- s3-sns: 500, 1000, 10k &rarr; 50k (increments of 5k)

**Note**: The `maestrina` is a convenience script to run all combinations using a single script. 
If you find any errors we recommend you to use the `antipode_lambda` script as described in the instructions.

After gathering all the results duplicate the `plots/configs/sample.yml` file into your configuration (e.g. `ae.yml`):
- Keep the `delay_vs_per_inconsistencies` as is
- Change the gather paths in `consistency_window` to the ones returned by the `gather` command by selecting only the results where the notification storage is `SNS`. In the end you will have a configuration similar to this:
```yml
consistency_window:
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
./plot plots/configs/ae.yml --plots delay_vs_per_inconsistencies
./plot plots/configs/ae.yml --plots consistency_window
```

### Antipode @ DeathStarBench

You should checkout the [`sosp23` release](https://github.com/Antipode-SOSP23/antipode-deathstarbench/tree/sosp23) and follow the instructions in that repo.
Next we provide a quick usage reference to get the main results from the paper. **Don't forget to check the pre-requisites and GCP configurations mentioned in instructions.**

```zsh
# Run maestrina for all combinations -- if you find errors use the maestro below
./maestrina

# For a single combination
./maestro --gcp socialNetwork build
./maestro --gcp socialNetwork deploy -config configs/gcp/socialNetwork/us-eu.yml -clients 1
./maestro --gcp socialNetwork run
./maestro --gcp socialNetwork wkld -E compose-post -d 300 -r 100  -c 4 -t 2
./maestro --gcp gather
```

For the full results obtained in the paper, you should run all the following combinations:
- Fix `clients` to 1, `connections` to 4 and `threads` to 2
- Run all rate values with and without Antipode (`-antipode` flag on `run`)
- Run the following rates: 50, 100, 125, 150, 160

After gathering all the results duplicate the `plots/configs/sample.yml` file into your configuration (e.g. `ae.yml`). 
Keep only the `throughput_latency_with_consistency_window` key, and add your gathered paths into that file, it should be similar to this:
```yml
throughput_latency_with_consistency_window:
  - gather/gcp/socialNetwork/compose-post/us-eu-50/20210814065624
  - gather/gcp/socialNetwork/compose-post/us-eu-100/20210814071451
  - gather/gcp/socialNetwork/compose-post/us-eu-125/20210814073301
  - gather/gcp/socialNetwork/compose-post/us-eu-150/20210814075114
  - gather/gcp/socialNetwork/compose-post/us-eu-160/20210814080933
  - gather/gcp/socialNetwork/compose-post/us-eu-50-antipode/20210814065624
  - gather/gcp/socialNetwork/compose-post/us-eu-100-antipode/20210814071451
  - gather/gcp/socialNetwork/compose-post/us-eu-125-antipode/20210814073301
  - gather/gcp/socialNetwork/compose-post/us-eu-150-antipode/20210814075114
  - gather/gcp/socialNetwork/compose-post/us-eu-160-antipode/20210814080933
```
_Note:_ Each workload should be ran more rounds (for the paper we used 15 rounds).


Now you just plot your results with:
```zsh
./plot plots/configs/ae.yml --plots throughput_latency_with_consistency_window
```


### Antipode @ Train-Ticket
You should checkout the [`sosp23` release](https://github.com/Antipode-SOSP23/antipode-train-ticket/tree/sosp23) and follow the instructions in that repo.
Next we provide a quick usage reference to get the main results from the paper. **Don't forget to check the pre-requisites and GCP configurations mentioned in instructions.**
```zsh
# Run maestrina for all combinations -- if you find errors use the maestro below
./maestrina

# For a single combination
./maestro --gcp build
./maestro --gcp deploy -config configs/gcp/socialNetwork/sosp23.yml -clients 1
./maestro --gcp run
./maestro --gcp wkld -d 300 -c 1 -t 8
./maestro --gcp gather
```

For the full results obtained in the paper, you should run all the following combinations:
- Fix `clients` to 1
- Run all thread values with and without Antipode (`-antipode` flag on `run`)
- Run the following threads: 1, 2, 3, 8, 10, 14

After gathering all the results duplicate the `plots/configs/sample.yml` file into your configuration (e.g. `ae.yml`). 
Keep only the `throughput_latency_with_consistency_window` key, and add your gathered paths into that file, it should be similar to this:
```yml
throughput_latency_with_consistency_window:
  - 20230720131918-1cli-1threads-antipode_no
  - 20230720143227-1cli-1threads-antipode_yes
  - 20230720131918-1cli-2threads-antipode_no
  - 20230720143227-1cli-2threads-antipode_yes
  - 20230720131918-1cli-3threads-antipode_no
  - 20230720143227-1cli-3threads-antipode_yes
  - 20230720131918-1cli-8threads-antipode_no
  - 20230720143227-1cli-8threads-antipode_yes
  - 20230720131918-1cli-10threads-antipode_no
  - 20230720143227-1cli-10threads-antipode_yes
  - 20230720131918-1cli-14threads-antipode_no
  - 20230720143227-1cli-14threads-antipode_yes
```

Now you just plot your results with:
```zsh
./plot plots/configs/ae.yml --plots throughput_latency_with_consistency_window
```

### Alibaba Spike
You should checkout the [`sosp23` release](https://github.com/Antipode-SOSP23/alibaba-spike/tree/sosp23) and follow the instructions in that repo.
Next we provide a quick usage reference to get the main results from the paper. **Don't forget to check the pre-requisites specially the Spark deployment.**
