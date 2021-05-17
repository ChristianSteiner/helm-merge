# helm merge value files
## How do I keep stage values out of my helm values yaml file

I've had the problem that I needed to have stage-specific values in my helm values file for my Prometheus deployment, but also had to 
keep them somewhat flexible.
I wanted to have one base values file for all my clusters, but have specific variables changed per cluster.

Specificaly I wanted to add extra labels for every scrape in my config and the labels should be "location", "project" and "stage".

## My solution: YAML Anchors and cat


I've split my values yaml in two pieces, the base.yaml with the whole config in it and the cluster/stage specific "prefix" file pre_cluster_stage.yml which only contains the changing variables.

In the base.yaml, I've added the new labels to every scrape job config as YAML Anchors:
```yaml  
  - job_name: 'kubernetes-nodes'
    ...
    relabel_configs:
          ...
          - action: replace
            target_label: location
            replacement: *custom_location      
          - action: replace
            target_label: project
            replacement: *custom_project    
          - action: replace
            target_label: stage
            replacement: *custom_stage
            
  - job_name: 'kubernetes-pods'
    ...
    relabel_configs:
          ...
          - action: replace
            target_label: location
            replacement: *custom_location      
          - action: replace
            target_label: project
            replacement: *custom_project    
          - action: replace
            target_label: stage
            replacement: *custom_stage
            
  - job_name: ...
```            
and so on.
            
In the cluster/stage specific prefix file, pre_project1_dev.yml, I've only added the definitons for the variables:
```yaml
  # set default values as YAML anchors
  custom1: &custom_location GCP
  custom2: &custom_project Project1
  custom3: &custom_stage DEV
```

Now I can combine both yaml files to one and use them for helm:
```bash
  helm upgrade --install prometheus prometheus-community/prometheus -f <(cat pre_project1_dev.yml base.yml)
```  
This combines both files and helm will use the parameters given in the pre file in its config.
