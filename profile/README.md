# Antipode

<picture>
  <img align="right" width="300" alt="Enso" src="https://github.com/Antipode-SOSP23/.github/assets/1867656/a2668e53-11ad-46f0-b30d-4d9a27871d3c">
</picture>

Antipode prevents **cross-service inconsistencies** in distributed applications by enforcing cross-service consistency. It propagates metadata both alongside end-to-end requests and within datastores. Antipode enables a novel **cross-service causal consistency** model, which extends existing causality models, and whose enforcement requires us to bring in a series of contributions to address fundamental semantic, scalability, and deployment challenges.

You can read our paper [Antipode: Enforcing Cross-Service Causal Consistency
in Distributed Applications](https://dl.acm.org/doi/10.1145/3600006.3613176) which was published at SOSP'23.

And you can quickly add it to your references:
```BibTeX
@inproceedings{loff2023antipode,
author = {Ferreira Loff, Jo\~{a}o and Porto, Daniel and Garcia, Jo\~{a}o and Mace, Jonathan and Rodrigues, Rodrigo},
title = {Antipode: Enforcing Cross-Service Causal Consistency in Distributed Applications},
year = {2023},
isbn = {9798400702297},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
url = {https://doi.org/10.1145/3600006.3613176},
doi = {10.1145/3600006.3613176},
abstract = {Modern internet-scale applications suffer from cross-service inconsistencies, arising because applications combine multiple independent and mutually-oblivious datastores. The end-to-end execution flow of each user request spans many different services and datastores along the way, implicitly establishing ordering dependencies among operations at different datastores. Readers should observe this ordering and, in today's systems, they do not.In this work, we present Antipode, a bolt-on technique for preventing cross-service consistency violations in distributed applications. It enforces cross-service consistency by propagating lineages of datastore operations both alongside end-to-end requests and within datastores. Antipode enables a novel cross-service causal consistency model, which extends existing causality models, and whose enforcement requires us to bring in a series of technical contributions to address fundamental semantic, scalability, and deployment challenges. We implemented Antipode as an application-level library, which can easily be integrated into existing applications with minimal effort, is incrementally deployable, and does not require global knowledge of all datastore operations. We apply Antipode to eight open-source and public cloud datastores and two microservice benchmark applications. Our evaluation demonstrates that Antipode is able to prevent cross-service inconsistencies with limited programming effort and less than 2\% impact on end-user latency and throughput.},
booktitle = {Proceedings of the 29th Symposium on Operating Systems Principles},
pages = {298–313},
numpages = {16},
location = {Koblenz, Germany},
series = {SOSP '23}
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

All scripts were run an Intel Core i7-6700K 4GHz with 8 cores and 16GB memory.


### Antipode @ Post-Notification

You should checkout the [`sosp23` release](https://github.com/Antipode-SOSP23/antipode-post-notification/tree/sosp23) and follow the instructions in that repo.
Next we provide a quick usage reference to get the main results from the paper. **Don't forget to check the pre-requisites and AWS configurations mentioned in instructions.**

```zsh
# Run maestrina for all combinations -- if you find errors use the antipode_lambda below
./maestrina

# For a single combination
./antipode_lambda build --post-storage mysql --notification-storage sns --writer eu --reader us
./antipode_lambda run -r 1000
./antipode_lambda gather
./antipode_lambda clean
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
- Set `clients` to 1, `connections` to 4 and `threads` to 2
- Run all rate values with and without Antipode (`-antipode` flag on `run`)
- Run the following rates: 50, 100, 125, 150, 160

After gathering all the results duplicate the `plots/configs/sample.yml` file into your configuration (e.g. `ae.yml`). 
Keep only the `throughput_latency_with_consistency_window` key, and add your gathered paths into that file, it should be similar to this (note that you need both `us-eu` and  `us-sg`):
```yml
throughput_latency_with_consistency_window:
  - gather/gcp/socialNetwork/compose-post/us-eu-50/20210814065624
  - gather/gcp/socialNetwork/compose-post/us-eu-50-antipode/20210814065624
  # ...
  - gather/gcp/socialNetwork/compose-post/us-sg-50/20210814065624
  - gather/gcp/socialNetwork/compose-post/us-sg-50-antipode/20210814065624
  # ...
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
./maestro --gcp wkld -d 300 -t 8
./maestro --gcp gather
```

For the full results obtained in the paper, you should run all the following combinations:
- Set `clients` to 1 (already done in the `deploy`)
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
  # ...
```

Now you just plot your results with:
```zsh
./plot plots/configs/ae.yml --plots throughput_latency_with_consistency_window
```

### Alibaba Spike
You should checkout the [`sosp23` release](https://github.com/Antipode-SOSP23/alibaba-spike/tree/sosp23) and follow the instructions in that repo.
Next we provide a quick usage reference to get the main results from the paper. **Don't forget to check the pre-requisites specially the Spark deployment.**

```zsh
./maestro --gsd deploy -inventory configs/gsd-inventory.yml -vars configs/gsd-vars.yml
./maestro --gsd stats # will generate a file inside the stats folder
./maestro --gsd plot -stats STATS_FILE_PATH -plots cdf_meta_rcptype_unique_services_and_calls # use the previous generated stats 
```
