https://blog.upweto.top/gitbooks/kubeSourceCodeNote/controller/

# scheduler 调度器  
![官方文档](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-scheduling/scheduler.md)

## 主要程序入口  
cmd/kube-scheduler/scheduler.go: This is the main() entry that does initialization before calling the scheduler framework.
pkg/scheduler/scheduler.go: This is the scheduler framework that handles stuff (e.g. binding) beyond the scheduling algorithm.
pkg/scheduler/core/generic_scheduler.go: The scheduling algorithm that assigns nodes for pods.

*默认算法配置*  pkg/scheduler/algorithmprovider/defaults/defaults.go   

## predicates 预选   
pkg/scheduler/algorithm/predicates/predicates.go
默认算法组合：  

## priority 优选    
pkg/scheduler/algorithm/priorities
打分0-10分,越大越优先;分数一样，随机选一个

修改默认策略
pkg/scheduler/algorithmprovider/defaults/defaults.go
指定启动参数： --policy-config-file

## 程序入口  
https://github.com/spf13/cobra  构建cli命令库

### 内部启动入口： pkg/scheduler/scheduler.go   
// scheduleOne does the entire scheduling workflow for a single pod.  It is serialized on the scheduling algorithm's host fitting.
func (sched *Scheduler) scheduleOne(ctx context.Context) { ... }
1.获取需调度的pod
2.使用调度算法寻找匹配node、发起绑定到node请求、绑定检查等一系列操作.
3.若匹配node失败，则尝试根据pod的指定优先级来抢占资源

pkg/scheduler/core/generic_scheduler.go
// Schedule tries to schedule the given pod to one of the nodes in the node list.
// If it succeeds, it will return the name of the node.
// If it fails, it will return a FitError error with reasons.
func (g *genericScheduler) Schedule(ctx context.Context, state *framework.CycleState, pod *v1.Pod) (result ScheduleResult, err error) { ... }
a. Run "prefilter" plugins.  
根据Predicate筛选node
filteredNodes, failedPredicateMap, filteredNodesStatuses, err := g.findNodesThatFit(ctx, state, pod)  
b.  Run "postfilter" plugins.  
// When only one node after predicate, just use it.    
c. 优选  
priorityList, err := g.prioritizeNodes(ctx, state, pod, metaPrioritiesInterface, filteredNodes)
host, err := g.selectHost(priorityList)  

```go
func (g *genericScheduler) selectHost(nodeScoreList framework.NodeScoreList) (string, error) {
	if len(nodeScoreList) == 0 {
		return "", fmt.Errorf("empty priorityList")
	}
	maxScore := nodeScoreList[0].Score
	selected := nodeScoreList[0].Name
	cntOfMaxScore := 1
	for _, ns := range nodeScoreList[1:] {
		if ns.Score > maxScore {
			maxScore = ns.Score
			selected = ns.Name
			cntOfMaxScore = 1
		} else if ns.Score == maxScore {
			cntOfMaxScore++
			if rand.Intn(cntOfMaxScore) == 0 {
				// Replace the candidate with probability of 1/cntOfMaxScore
				selected = ns.Name
			}
		}
	}
	return selected, nil
}
```

### 调度器框架
pkg\scheduler\algorithmprovider\defaults\defaults.go

1. RegisterAlgorithmProvider： SchedulerDefaultProviderName  
func registerAlgorithmProvider(predSet, priSet sets.String) {
	// Registers algorithm providers. By default we use 'DefaultProvider', but user can specify one to be used
	// by specifying flag.
	scheduler.RegisterAlgorithmProvider(schedulerapi.SchedulerDefaultProviderName, predSet, priSet)
	// Cluster autoscaler friendly scheduling algorithm.
	scheduler.RegisterAlgorithmProvider(ClusterAutoscalerProvider, predSet,
		copyAndReplace(priSet, priorities.LeastRequestedPriority, priorities.MostRequestedPriority))
}
2. pkg\scheduler\algorithm_factory.go   绑定配置 predicateKeys, priorityKeys
// RegisterAlgorithmProvider registers a new algorithm provider with the algorithm registry.
func RegisterAlgorithmProvider(name string, predicateKeys, priorityKeys sets.String) string {
	schedulerFactoryMutex.Lock()
	defer schedulerFactoryMutex.Unlock()
	validateAlgorithmNameOrDie(name)
	algorithmProviderMap[name] = AlgorithmProviderConfig{
		FitPredicateKeys:     predicateKeys,
		PriorityFunctionKeys: priorityKeys,
	}
	return name
}  

3. pkg/scheduler/algorithmprovider/defaults/register_predicates.go,register_priorities.go 具体实现
* RegisterFitPredicate  
// Fit is determined by node selector query.
	scheduler.RegisterFitPredicate(predicates.MatchNodeSelectorPred,predicates.PodMatchNodeSelector)

pkg\scheduler\algorithm_factory.go
```go
// RegisterFitPredicate registers a fit predicate with the algorithm
// registry. Returns the name with which the predicate was registered.
func RegisterFitPredicate(name string, predicate predicates.FitPredicate) string {
	return RegisterFitPredicateFactory(name, func(AlgorithmFactoryArgs) predicates.FitPredicate { return predicate })
}
// RegisterFitPredicateFactory registers a fit predicate factory with the
// algorithm registry. Returns the name with which the predicate was registered.
func RegisterFitPredicateFactory(name string, predicateFactory FitPredicateFactory) string {
	schedulerFactoryMutex.Lock()
	defer schedulerFactoryMutex.Unlock()
	validateAlgorithmNameOrDie(name)
	fitPredicateMap[name] = predicateFactory
	return name
}
```
4. 生成默认配置  
cmd\kube-scheduler\app\options\options.go： kubeschedulerscheme.Scheme.Default
func newDefaultComponentConfig() (*kubeschedulerconfig.KubeSchedulerConfiguration, error) {
	cfgv1alpha1 := kubeschedulerconfigv1alpha1.KubeSchedulerConfiguration{}
	kubeschedulerscheme.Scheme.Default(&cfgv1alpha1)
	cfg := kubeschedulerconfig.KubeSchedulerConfiguration{}
	if err := kubeschedulerscheme.Scheme.Convert(&cfgv1alpha1, &cfg, nil); err != nil {
		return nil, err
	}
	return &cfg, nil
}
引用 pkg\scheduler\apis\config\scheme\scheme.go  

func (s *Scheme) AddTypeDefaultingFunc(srcType Object, fn func(interface{})) {
    s.defaulterFuncs[reflect.TypeOf(srcType)] = fn
}

pkg\scheduler\apis\config\v1alpha1\defaults.go (SchedulerDefaultProviderName)
// SetDefaults_KubeSchedulerConfiguration sets additional defaults
func SetDefaults_KubeSchedulerConfiguration(obj *kubeschedulerconfigv1alpha1.KubeSchedulerConfiguration) {
	if obj.SchedulerName == nil {
		val := api.DefaultSchedulerName
		obj.SchedulerName = &val
	}
    .....
	if obj.AlgorithmSource.Policy == nil &&
		(obj.AlgorithmSource.Provider == nil || len(*obj.AlgorithmSource.Provider) == 0) {
		val := kubeschedulerconfigv1alpha1.SchedulerDefaultProviderName
		obj.AlgorithmSource.Provider = &val
	}
    .......
}

### Node预算算法  
 pkg\scheduler\core\generic_scheduler.go  
 // Filters the nodes to find the ones that fit based on the given predicate functions
// Each node is passed through the predicate functions to determine if it is a fit
func (g *genericScheduler) findNodesThatFit(ctx context.Context, state *framework.CycleState, pod *v1.Pod) ([]*v1.Node, FailedPredicateMap, framework.NodeToStatusMap, error) {
    ...
    // 筛选的node对象的数量，点击进去可查看详情，当集群规模小于100台时，全部检查，当集群大于100台时，
    // 检查指定比例的机器，若指定比例范围内都没有找到合适的node，则继续查找
        numNodesToFind := g.numFeasibleNodesToFind(allNodes)

    // Stops searching for more nodes once the configured number of feasible nodes are found.
		workqueue.ParallelizeUntil(ctx, 16, allNodes, checkNode)
}

// numFeasibleNodesToFind returns the number of feasible nodes that once found, the scheduler stops
// its search for more feasible nodes.
func (g *genericScheduler) numFeasibleNodesToFind(numAllNodes int32) (numNodes int32) {
	if numAllNodes < minFeasibleNodesToFind || g.percentageOfNodesToScore >= 100 {
		return numAllNodes
	}

	adaptivePercentage := g.percentageOfNodesToScore
	if adaptivePercentage <= 0 {
		basePercentageOfNodesToScore := int32(50)
		adaptivePercentage = basePercentageOfNodesToScore - numAllNodes/125
		if adaptivePercentage < minFeasibleNodesPercentageToFind {
			adaptivePercentage = minFeasibleNodesPercentageToFind
		}
	}

	numNodes = numAllNodes * adaptivePercentage / 100
	if numNodes < minFeasibleNodesToFind {
		return minFeasibleNodesToFind
	}

	return numNodes
}

并发： staging\src\k8s.io\client-go\util\workqueue\parallelizer.go 
pkg\scheduler\internal\cache\node_tree.go  

对于每个提名pod,其调度过程会被重复执行1次，为什么需要重复执行呢？考虑到有一些场景下，会判断到pod之间的亲和力筛选策略，例如pod A对pod B有亲和性，这时它们一起调度到node上，但pod B此时实际并未完成调度启动，那么pod A的inter-pod affinity predicates一定会失败，因此，重复执行1次筛选过程是有必要的

pkg\scheduler\algorithm\predicates\predicates.go  算法明细  