## What problems does the AWS Fargate container level metrics try to solve?

The metrics collected by Containers Insights for ECS (which includes support for Fargate) isn't granular enough to allow tracking single containers inside individual tasks. In some circumstances it is helpful to get into the weeds of how single containers behave inside specific tasks. Example includes being able to fine tune or introduce container-level reservations and caps.

How can we go one level down deep from the task definition family (as provided by default by Container Insights) or from the task itself (as provided by the [Fargate right sizing dashboard](../fargate-right-sizing)) into each single running Fargate task? Enter the Fargate container level metrics dashboard.

## How does the AWS Fargate container level metrics dashboard look like?

![container-level-metrics](../images/container-level-metrics.png)  

The Fargate container level metrics dashboard uses CloudWatch Logs Insights to scan and analyze performance logs collected from the cluster you want to optimize.

The dashboard tries to respond to these user stories (as a user I would like to):
- see the list of containers that are part of the task I am inspecting
- for all containers, identify the cpu reservation (if any), the average cpu load and the peak cpu load
- for all containers, identify the memory reservation (if any), the average memory load and the peak memory load
- for the selected container, visualize the cpu reservation (if any) and the average cpu consumption over time
- for the selected container, visualize the memory reservation (if any) and the average memory consumption over time
- as a reference, see the performance characteristics of the task I am inspecting (cpu and memory load, reservations, peaks and more)

## How do I import the dashboard?

To import the dashboard clone this repo and move into the `container-level-metrics` directory:

```
git clone https://github.com/mreferre/container-insights-custom-dashboards.git
cd container-insights-custom-dashboards/container-level-metrics
```

At this point you will need to configure the source of each widget to point to the log group for the cluster you intend to track. For example, for an ECS cluster named `cluster-prod` that has been configured to use CW Container Insights, there will be a log group called `/aws/ecs/containerinsights/cluster-prod/performance`.

This log group needs to replace the placeholder log group in the `container-level-metrics.json` file. The placeholder in the file is `/aws/ecs/containerinsights/CLUSTERNAME/performance`. 

You can use `sed`:
```
# GNU sed (linux)
sed -i 's/CLUSTERNAME/cluster-prod/g' container-level-metrics.json

# BSD sed (mac)
sed -i.bak 's/CLUSTERNAME/cluster-prod/g' container-level-metrics.json
```

This also needs to be done with the name of the task you want to examine and the name of the container you are interested in. The placeholder for the container name is `CONTAINERNAMEREPLACEME` and the placeholder for the task id is `TASKIDREPLACEME`.

```
# GNU sed (linux)
sed -i 's/CONTAINERNAMEREPLACEME/<my container name>/g' container-level-metrics.json
sed -i 's/TASKIDREPLACEME/<my task id>/g' container-level-metrics.json

# BSD sed (mac)
sed -i.bak 's/CONTAINERNAMEREPLACEME/<my container name>/g' container-level-metrics.json
sed -i.bak 's/TASKIDREPLACEME/<my task id>/g' container-level-metrics.json
```

Now you are ready to import the dashboard with the following command:
```
aws cloudwatch put-dashboard --dashboard-name container-level-metrics --dashboard-body file://./container-level-metrics.json
```

If you want/need to examine other containers and tasks, you can edit the dashboard and modify the task id and container name. 

## Known issues and limitations

- This dashboard only tracks and consider ECS/Fargate tasks. It doesn't consider ECS/EC2 tasks (because the optimization considerations for tasks running on EC2 may, possibly, be very different due to sharing of resources and over-commitment capabilities)
- All tasks are considered for the period you specified. That is, this includes also tasks that are no longer running 
- Creating % utilization based graphs isn't trivial because it depends on how the task and the containers are configured. Containers may have different ceilings depending on whether reservation is specified. Because of this, this dashboard does not provide a percent-based consumption value  
- The default retention of the Container Insights performance logs is 1 day. This means that by default the graphs can only track the previous 24 hours
- If you want these data to persist you can change manually the retention period of the cluster CloudWatch log group
- These are logs and not metrics. Hence, you cannot set alarms like you'd normally do with metrics
