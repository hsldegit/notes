# Spring Cloud微服务架构进阶
<!-- TOC -->
[toc]
<!-- /TOC -->
## 微服务架构的出现

微服务最早是由Martin Fowler与James Lewis于2014年共同提出其实Martin先生并没有给出明确的微服务定义，根据其描述，微服务的定义可以概括如下：微服务架构是一种使用一系列粒度较小的服务来开发单个应用的方式；每个服务运行在自己的进程中；服务间采用轻量级的方式进行通信(通常是HTTP API)；这些服务是基于业务逻辑和范围，通过自动化部署的机制来独立部署的，并且服务的集中管理应该是最低限度的，即每个服务可以采用不同的编程语言编写，使用不同的数据存储技术。

微服务架构是一种比较复杂、内涵丰富的架构模式，它包含很多支撑“微”服务的具体组件和概念，其中一些常用的组件及其概念如下：

- 服务注册与发现：服务提供方将己方调用地址注册到服务注册中心，让服务调用方能够方便地找到自己；服务调用方从服务注册中心找到自己需要调用的服务的地址。
- 负载均衡：服务提供方一般以多实例的形式提供服务，负载均衡功能能够让服务调用方连接到合适的服务节点。并且，服务节点选择的过程对服务调用方来说是透明的。
- 服务网关：服务网关是服务调用的唯一入口，可以在这个组件中实现用户鉴权、动态路由、灰度发布、A/B测试、负载限流等功能。
- 配置中心：将本地化的配置信息(Properties、XML、YAML等形式)注册到配置中心，实现程序包在开发、测试、生产环境中的无差别性，方便程序包的迁移。
- 集成框架：微服务组件都以职责单一的程序包对外提供服务，集成框架以配置的形式将所有微服务组件(特别是管理端组件)集成到统一的界面框架下，让用户能够在统一的界面中使用系统。
- 调用链监控：记录完成一次请求的先后衔接和调用关系，并将这种串行或并行的调用关系展示出来。在系统出错时，可以方便地找到出错点。
- 支撑平台：系统微服务化后，各个业务模块经过拆分变得更加细化，系统的部署、运维、监控等都比单体应用架构更加复杂，这就需要将大部分的工作自动化。现在，Docker等工具可以给微服务架构的部署带来较多的便利，例如持续集成、蓝绿发布、健康检查、性能健康等等。如果没有合适的支撑平台或工具，微服务架构就无法发挥它最大的功效。

## SpringCloud总览

在介绍微服务架构相关的概念之后，本章将会介绍Spring Cloud相关的概念。Spring Cloud是一系列框架的有机集合，基于Spring Boot实现的云应用开发工具，为云原生应用开发中的服务发现与注册、熔断机制、服务路由、分布式配置中心、消息总线、负载均衡和链路监控等功能的实现提供了一种简单的开发方式。

### SpringCloud架构

Spring Cloud包含的组件众多，各个组件都有各自不同的特色和优点，为用户提供丰富的选择：

- 服务注册与发现组件：Eureka、Zookeeper和Consul等。本书将会重点讲解Eureka，Eureka是一个REST风格的服务注册与发现的基础服务组件。
- 服务调用组件：Hystrix、Ribbon和OpenFeign；其中Hystrix能够使系统在出现依赖服务失效的情况下，通过隔离系统依赖服务的方式，防止服务级联失败，同时提供失败回滚机制，使系统能够更快地从异常中恢复；Ribbon用于提供客户端的软件负载均衡算法，还提供了一系列完善的配置项如连接超时、重试等；OpenFeign是一个声明式RESTful网络请求客户端，它使编写Web服务客户端变得更加方便和快捷。
- 配置中心组件：Spring Cloud Config实现了配置集中管理、动态刷新等配置中心的功能。配置通过Git或者简单文件来存储，支持加解密。
- 消息组件：Spring Cloud Stream和Spring Cloud Bus。Spring Cloud Stream对于分布式消息的各种需求进行了抽象，包括发布订阅、分组消费和消息分区等功能，实现了微服务之间的异步通信。Spring Cloud Bus主要提供了服务间的事件通信(如刷新配置)。
- 安全控制组件：Spring Cloud Security基于OAuth2.0开放网络的安全标准，提供了微服务环境下的单点登录、资源授权和令牌管理等功能。
- 链路监控组件：Spring Cloud Sleuth提供了全自动、可配置的数据埋点，以收集微服务调用链路上的性能数据，并可以结合Zipkin进行数据存储、统计和展示。

## Spring Boot

1111

## 注册与发现服务

### Eureka Client

为了对Eureka Client的执行原理进行讲解，首先需要对服务发现客户端com.netflix.discover.DiscoveryClient职能以及相关类进行讲解，它负责了与Eureka Server交互的关键逻辑

#### DiscoveryClient

DiscoveryClient的构造函数中，主要依次做了以下的事情:

)相关配置的赋值，类似ApplicationInfoManager、EurekaClientConfig等。
2)备份注册中心的初始化，默认没有实现。
3)拉取Eureka Server注册表中的信息。
)注册前的预处理。
5)向Eureka Server注册自身。
6)初始化心跳定时任务、缓存刷新和按需注册等定时任务。(2个线程 1个心跳 一个刷新缓存)

在DiscoveryClient的构造函数中，调用了DiscoveryClient#fetchRegistry方法从Eureka Server中拉取注册表信息

```java
    private boolean fetchRegistry(boolean forceFullRegistryFetch) {
        Stopwatch tracer = FETCH_REGISTRY_TIMER.start();

        try {
            // If the delta is disabled or if it is the first time, get all
            // applications
            Applications applications = getApplications();

            if (clientConfig.shouldDisableDelta()
                    || (!Strings.isNullOrEmpty(clientConfig.getRegistryRefreshSingleVipAddress()))
                    || forceFullRegistryFetch
                    || (applications == null)
                    || (applications.getRegisteredApplications().size() == 0)
                    || (applications.getVersion() == -1)) //Client application does not have latest library supporting delta
            {
                logger.info("Disable delta property : {}", clientConfig.shouldDisableDelta());
                logger.info("Single vip registry refresh property : {}", clientConfig.getRegistryRefreshSingleVipAddress());
                logger.info("Force full registry fetch : {}", forceFullRegistryFetch);
                logger.info("Application is null : {}", (applications == null));
                logger.info("Registered Applications size is zero : {}",
                        (applications.getRegisteredApplications().size() == 0));
                logger.info("Application version is -1: {}", (applications.getVersion() == -1));
                //全量拉取注册信息
                getAndStoreFullRegistry();
            } else {
                //增量拉取
                getAndUpdateDelta(applications);
            }
            applications.setAppsHashCode(applications.getReconcileHashCode());
            logTotalInstances();
        } catch (Throwable e) {
            logger.error(PREFIX + "{} - was unable to refresh its cache! status = {}", appPathIdentifier, e.getMessage(), e);
            return false;
        } finally {
            if (tracer != null) {
                tracer.stop();
            }
        }
        // 在更新远程实例状态之前推送缓存刷新事件，但是Eureka中并没有提供默认的事件监听器
        // Notify about cache refresh before updating the instance remote status
        onCacheRefreshed();

        // Update remote status based on refreshed data held in the cache
        // 基于缓存中被刷新的数据更新远程实例状态
        updateInstanceRemoteStatus();

        // registry was fetched successfully, so return true
        return true;
    }
```

全量拉取注册信息

```java
private void getAndStoreFullRegistry() throws Throwable {
        //获取领取注册表的版本防止拉取版本落后
        long currentUpdateGeneration = fetchRegistryGeneration.get();
        logger.info("Getting all instance registry info from the eureka server");

        Applications apps = null;
        EurekaHttpResponse<Applications> httpResponse = clientConfig.getRegistryRefreshSingleVipAddress() == null
                ? eurekaTransport.queryClient.getApplications(remoteRegionsRef.get())
                : eurekaTransport.queryClient.getVip(clientConfig.getRegistryRefreshSingleVipAddress(), remoteRegionsRef.get());
                //获取成功
        if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
            apps = httpResponse.getEntity();
        }
        logger.info("The response status is {}", httpResponse.getStatusCode());

        if (apps == null) {
            logger.error("The application is null for some reason. Not storing this information");
        } else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
            localRegionApps.set(this.filterAndShuffle(apps));
            logger.debug("Got full registry with apps hashcode {}", apps.getAppsHashCode());
        } else {
            logger.warn("Not updating applications as another thread is updating it already");
        }
    }
```

getAndStoreFullRegistry方法可能被多个线程同时调用，导致新拉取的注册表被旧的注册表覆盖(有可能出现先拉取注册表信息的线程在覆盖apps时被阻塞，被后拉取注册表信息的线程抢先设置了apps，被阻塞的线程恢复后再次设置了apps，导致apps数据版本落后)，产生脏数据，对此，Eureka通过类型为AtomicLong的currentUpdateGeneration对apps的更新版本进行跟踪。如果更新版本不一致，说明本次拉取注册表信息已过时，不需要缓存到本地。拉取到注册表信息之后会对获取到的apps进行筛选，只保留状态为UP的服务实例信息。

增量式拉取注册表信息
增量式的拉取方式，一般发生在第一次拉取注册表信息之后，拉取的信息定义为从某一段时间之后发生的所有变更信息，通常来讲是3分钟之内注册表的信息变化。在获取到更新的delta后，会根据delta中的增量更新对本地的数据进行更新。与getAndStoreFullRegistry方法一样，也通过fetchRegistryGeneration对更新的版本进行控制。增量式拉取是为了维护Eureka Client本地的注册表信息与Eureka Server注册表信息的一致性，防止数据过久而失效，采用增量式拉取的方式减少了拉取注册表信息的通信量。Client中有一个注册表缓存刷新定时器专门负责维护两者之间信息的同步性。但是当增量式拉取出现意外时，定时器将执行全量拉取以更新本地缓存的注册表信息。

```java
    private void getAndUpdateDelta(Applications applications) throws Throwable {
        long currentUpdateGeneration = fetchRegistryGeneration.get();

        Applications delta = null;
        EurekaHttpResponse<Applications> httpResponse = eurekaTransport.queryClient.getDelta(remoteRegionsRef.get());
        if (httpResponse.getStatusCode() == Status.OK.getStatusCode()) {
            delta = httpResponse.getEntity();
        }
        // 获取增量拉取失败
        if (delta == null) {
            logger.warn("The server does not allow the delta revision to be applied because it is not safe. "
                    + "Hence got the full registry.");
                    //进行全量拉取
            getAndStoreFullRegistry();
        } else if (fetchRegistryGeneration.compareAndSet(currentUpdateGeneration, currentUpdateGeneration + 1)) {
            logger.debug("Got delta update with apps hashcode {}", delta.getAppsHashCode());
            String reconcileHashCode = "";
            if (fetchRegistryUpdateLock.tryLock()) {
                try {
                    // 更新本地缓存
                    updateDelta(delta);
                    // 计算应用集合一致性哈希码
                    reconcileHashCode = getReconcileHashCode(applications);
                } finally {
                    fetchRegistryUpdateLock.unlock();
                }
            } else {
                logger.warn("Cannot acquire update lock, aborting getAndUpdateDelta");
            }
            // 比较应用集合一致性哈希码，如果不一致将认为本次增量式拉取数据已脏，将发起全量拉取更新本地注册表信息
            // There is a diff in number of instances for some reason
            if (!reconcileHashCode.equals(delta.getAppsHashCode()) || clientConfig.shouldLogDeltaDiff()) {
                reconcileAndLogDifference(delta, reconcileHashCode);  // this makes a remoteCall
            }
        } else {
            logger.warn("Not updating application delta as another thread is updating it already");
            logger.debug("Ignoring delta update with apps hashcode {}, as another thread is updating it already", delta.getAppsHashCode());
        }
    }
```

在根据从Eureka Server拉取的delta信息更新本地缓存的时候，Eureka定义了ActionType来标记变更状态，代码位于InstanceInfo类中，如下所示：

```java
// InstanceInfo.java
public enum ActionType {
    ADDED,　　// 添加Eureka Server
    MODIFIED, // 在Euerka Server中的信息发生改变
    DELETED　 // 被从Eureka Server中剔除
}

```

具体操作

```java
    private void updateDelta(Applications delta) {
        int deltaCount = 0;
        for (Application app : delta.getRegisteredApplications()) {
            for (InstanceInfo instance : app.getInstances()) {
                Applications applications = getApplications();
                String instanceRegion = instanceRegionChecker.getInstanceRegion(instance);
                if (!instanceRegionChecker.isLocalRegion(instanceRegion)) {
                    Applications remoteApps = remoteRegionVsApps.get(instanceRegion);
                    if (null == remoteApps) {
                        remoteApps = new Applications();
                        remoteRegionVsApps.put(instanceRegion, remoteApps);
                    }
                    applications = remoteApps;
                }

                ++deltaCount;
                if (ActionType.ADDED.equals(instance.getActionType())) {
                    Application existingApp = applications.getRegisteredApplications(instance.getAppName());
                    if (existingApp == null) {
                        applications.addApplication(app);
                    }
                    logger.debug("Added instance {} to the existing apps in region {}", instance.getId(), instanceRegion);
                    //添加到本地注册表
                    applications.getRegisteredApplications(instance.getAppName()).addInstance(instance);
                } else if (ActionType.MODIFIED.equals(instance.getActionType())) {
                    Application existingApp = applications.getRegisteredApplications(instance.getAppName());
                    if (existingApp == null) {
                        applications.addApplication(app);
                    }
                    logger.debug("Modified instance {} to the existing apps ", instance.getId());

                    applications.getRegisteredApplications(instance.getAppName()).addInstance(instance);

                } else if (ActionType.DELETED.equals(instance.getActionType())) {
                    Application existingApp = applications.getRegisteredApplications(instance.getAppName());
                    if (existingApp == null) {
                        applications.addApplication(app);
                    }
                    logger.debug("Deleted instance {} to the existing apps ", instance.getId());
                    applications.getRegisteredApplications(instance.getAppName()).removeInstance(instance);
                }
            }
        }
        logger.debug("The total number of instances fetched by the delta processor : {}", deltaCount);

        getApplications().setVersion(delta.getVersion());
        getApplications().shuffleInstances(clientConfig.shouldFilterOnlyUpInstances());

        for (Applications applications : remoteRegionVsApps.values()) {
            applications.setVersion(delta.getVersion());
            applications.shuffleInstances(clientConfig.shouldFilterOnlyUpInstances());
        }
    }
```

更新本地注册表缓存之后，Eureka Client会通过#getReconcileHashCode计算合并后的Applications的appsHashCode(应用集合一致性哈希码)，和Eureka Server传递的delta上的appsHashCode进行比较(delta中携带的appsHashCode通过Eureka Server的全量注册表计算得出)，比对客户端和服务端上注册表的差异。如果哈希值不一致的话将再调用一次getAndStoreFullRegistry获取全量数据保证Eureka Client与Eureka Server之间注册表数据的一致。

在拉取完Eureka Server中的注册表信息并将其缓存在本地后，Eureka Client将向Eureka Server注册自身服务实例元数据，主要逻辑位于Discovery#register方法中。

```java
boolean register() throws Throwable {
    EurekaHttpResponse〈Void〉 httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        throw e;
    }
    ...
    // 注册成功
    return httpResponse.getStatusCode() == 204;
}
```

初始化定时任务

```java
// DiscoveryClient.java
private void initScheduledTasks() {
    if (clientConfig.shouldFetchRegistry()) {
        // 注册表缓存刷新定时器
        // 获取配置文件中刷新间隔，默认为30s，可以通过eureka.client.registry-fetch-
                interval-seconds进行设置
        int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();

                int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
        int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound(); scheduler.schedule(
            new TimedSupervisorTask("cacheRefresh", scheduler, cacheRefreshExecutor,
                registryFetchIntervalSeconds, TimeUnit.SECONDS, expBackOffBound, new CacheRefreshThread()
            ),
            registryFetchIntervalSeconds, TimeUnit.SECONDS);
    }
    if (clientConfig.shouldRegisterWithEureka()) {
        // 发送心跳定时器，默认30秒发送一次心跳
        int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
        int expBackOffBound = clientConfig.getHeartbeatExecutorExponentialBackOffBound();
       // 心跳定时器
        scheduler.schedule(
            new TimedSupervisorTask("heartbeat", scheduler, heartbeatExecutor,
                renewalIntervalInSecs,
                TimeUnit.SECONDS, expBackOffBound, new HeartbeatThread()
            ),
            renewalIntervalInSecs, TimeUnit.SECONDS);
        // 按需注册定时器
        ...
}
```
服务下线

```java
// DiscoveryClient.java
@PreDestroy
@Override
public synchronized void shutdown() {
    // 同步方法
    if (isShutdown.compareAndSet(false, true)) {
        // 原子操作，确保只会执行一次
        if (statusChangeListener != null &amp;&amp; applicationInfoManager != null) {
        // 注销状态监听器
    applicationInfoManager.unregisterStatusChangeListener(statusChangeListener.getId());
        }
        // 取消定时任务
        cancelScheduledTasks();
        if (applicationInfoManager != null &amp;&amp; clientConfig.shouldRegisterWithEureka()) {
            // 服务下线
            applicationInfoManager.setInstanceStatus(InstanceStatus.DOWN);
            unregister();
        }
        // 关闭Jersy客户端
        if (eurekaTransport != null) {
            eurekaTransport.shutdown();
        }
        // 关闭相关Monitor
        heartbeatStalenessMonitor.shutdown();
        registryStalenessMonitor.shutdown();
    }
}
```

### Eureka Server源码解析

Eureka Server作为一个开箱即用的服务注册中心，提供了以下的功能，用以满足与Eureka Client交互的需求：

- 服务注册
- 接受服务心跳
- 服务剔除
- 服务下线
- 集群同步
- 获取注册表中服务实例信息

需要注意的是，Eureka Server同时也是一个Eureka Client，在不禁止Eureka Server的客户端行为时，它会向它配置文件中的其他Eureka Server进行拉取注册表、服务注册和发送心跳等操作。

服务注册 我们了解到Eureka Client在发起服务注册时会将自身的服务实例元数据封装在InstanceInfo中，然后将InstanceInfo发送到Eureka Server。Eureka Server在接收到Eureka Client发送的InstanceInfo后将会尝试将其放到本地注册表中以供其他Eureka Client进行服务发现。
在register中，服务实例的InstanceInfo保存在Lease中，Lease在AbstractInstanceRegistry中统一通过ConcurrentHashMap保存在内存中。在服务注册过程中，会先获取一个读锁，防止其他线程对registry注册表进行数据操作，避免数据的不一致。然后从resgitry查询对应的InstanceInfo租约是否已经存在注册表中，根据appName划分服务集群，使用InstanceId唯一标记服务实例。如果租约存在，比较两个租约中的InstanceInfo的最后更新时间lastDirtyTimestamp，保留时间戳大的服务实例信息InstanceInfo。如果租约不存在，意味这是一次全新的服务注册，将会进行自我保护的统计，创建新的租约保存InstanceInfo。接着将租约放到resgitry注册表中。
之后将进行一系列缓存操作并根据覆盖状态规则设置服务实例的状态，缓存操作包括将InstanceInfo加入用于统计Eureka Client增量式获取注册表信息的recentlyChangedQueue和失效responseCache中对应的缓存。最后设置服务实例租约的上线时间用于计算租约的有效时间，释放读锁并完成服务注册。代码如下所示：

```java
//AbstractInstanceRegistry.java
 public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
        try {
            //获取读锁
            read.lock();
            // 这里的registry是ConcurrentHashMap〈String, Map〈String, Lease〈InstanceInfo〉〉〉registry，根据appName对服务实例集群进行分类
            Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
            REGISTER.increment(isReplication);
            if (gMap == null) {
                final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
                // 这里有一个比较严谨的操作，防止在添加新的服务实例集群租约时把已有的其他线程添加的集群租约覆盖掉，如果存在该键值，直接返回已存在的值；否则添加该键值对，返回null
                gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
                if (gMap == null) {
                    gMap = gNewMap;
                }
            }
            //根据instanceId获取实例的租约
            Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
            // Retain the last dirty timestamp without overwriting it, if there is already a lease
            if (existingLease != null && (existingLease.getHolder() != null)) {
                Long existingLastDirtyTimestamp = existingLease.getHolder().getLastDirtyTimestamp();
                Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();
                logger.debug("Existing lease found (existing={}, provided={}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);

                // this is a > instead of a >= because if the timestamps are equal, we still take the remote transmitted
                // InstanceInfo instead of the server local copy.
                // 如果该实例的租约已经存在，比较最后更新时间戳的大小，取最大值的注册信息为有效
                if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
                    logger.warn("There is an existing lease and the existing lease's dirty timestamp {} is greater" +
                            " than the one that is being registered {}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                    logger.warn("Using the existing instanceInfo instead of the new instanceInfo as the registrant");
                    registrant = existingLease.getHolder();
                }
            } else {
                // The lease does not exist and hence it is a new registration
                // 果租约不存在，这是一个新的注册实例
                synchronized (lock) {
                    if (this.expectedNumberOfRenewsPerMin > 0) {
                        // Since the client wants to cancel it, reduce the threshold
                        // (1
                        // for 30 seconds, 2 for a minute)
                        // 自我保护机制
                        this.expectedNumberOfRenewsPerMin = this.expectedNumberOfRenewsPerMin + 2;
                        this.numberOfRenewsPerMinThreshold =
                                (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
                    }
                }
                logger.debug("No previous lease information found; it is new registration");
            }
            // 创建新的租约
            Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
            if (existingLease != null) {
                // 如果租约存在，继承租约的服务上线初始时间
                lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
            }
            // 保存租约
            gMap.put(registrant.getId(), lease);
            // 添加最近注册队列
            synchronized (recentRegisteredQueue) {
                recentRegisteredQueue.add(new Pair<Long, String>(
                        System.currentTimeMillis(),
                        registrant.getAppName() + "(" + registrant.getId() + ")"));
            }
            // This is where the initial state transfer of overridden status happens
            if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
                logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the "
                                + "overrides", registrant.getOverriddenStatus(), registrant.getId());
                if (!overriddenInstanceStatusMap.containsKey(registrant.getId())) {
                    logger.info("Not found overridden id {} and hence adding it", registrant.getId());
                    overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
                }
            }
            InstanceStatus overriddenStatusFromMap = overriddenInstanceStatusMap.get(registrant.getId());
            if (overriddenStatusFromMap != null) {
                logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
                registrant.setOverriddenStatus(overriddenStatusFromMap);
            }

            // Set the status based on the overridden status rules
             // 根据覆盖状态规则得到服务实例的最终状态，并设置服务实例的当前状态
            InstanceStatus overriddenInstanceStatus = getOverriddenInstanceStatus(registrant, existingLease, isReplication);
            registrant.setStatusWithoutDirty(overriddenInstanceStatus);

            // If the lease is registered with UP status, set lease service up timestamp
            // 如果服务实例状态为UP，设置租约的服务上线时间，只有第一次设置有效
            if (InstanceStatus.UP.equals(registrant.getStatus())) {
                lease.serviceUp();
            }
            // 添加最近租约变更记录队列，标识ActionType为ADDED
            registrant.setActionType(ActionType.ADDED);
            // 这将用于Eureka Client增量式获取注册表信息
            recentlyChangedQueue.add(new RecentlyChangedItem(lease));
            // 设置服务实例信息更新时间
            registrant.setLastUpdatedTimestamp();
            // 设置response缓存过期，这将用于Eureka Client全量获取注册表信息
            invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
            logger.info("Registered instance {}/{} with status {} (replication={})",
                    registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication);
        } finally {
            read.unlock();
        }
    }
```

接受服务心跳

Eureka Client完成服务注册之后，它需要定时向Eureka Server发送心跳请求(默认30秒一次)，维持自己在Eureka Server中租约的有效性。
Eureka Server处理心跳请求的核心逻辑位于AbstractInstanceRegistry#renew方法中。renew方法是对Eureka Client位于注册表中的租约的续租操作，不像register方法需要服务实例信息，仅根据服务实例的服务名和服务实例id即可更新对应租约的有效时间。具体代码如下所示：

```java
// AbstractInstanceRegistry.java
public boolean renew(String appName, String id, boolean isReplication) {
    RENEW.increment(isReplication);
    // 根据appName获取服务集群的租约集合
    Map〈String, Lease〈InstanceInfo〉〉 gMap = registry.get(appName);
    Lease〈InstanceInfo〉 leaseToRenew = null;
    if (gMap != null) {
    leaseToRenew = gMap.get(id);
    }
    // 租约不存在，直接返回false
    if (leaseToRenew == null) {
        RENEW_NOT_FOUND.increment(isReplication);
        return false;
    } else {
        InstanceInfo instanceInfo = leaseToRenew.getHolder();
        if (instanceInfo != null) {
           // 根据覆盖状态规则得到服务实例的最终状态
            InstanceStatus overriddenInstanceStatus = this.getOverriddenInstanceStatus(instanceInfo, leaseToRenew, isReplication);
            if (overriddenInstanceStatus == InstanceStatus.UNKNOWN) {
                // 如果得到的服务实例最后状态是UNKNOWN，取消续约
                RENEW_NOT_FOUND.increment(isReplication);
                return false;
            }
            if (!instanceInfo.getStatus().equals(overriddenInstanceStatus)) {
                instanceInfo.setStatus(overriddenInstanceStatus);
            }
        }
        // 统计每分钟续租的次数，用于自我保护
        renewsLastMin.increment();
        // 更新租约中的有效时间
        leaseToRenew.renew();
        return true;
    }
}
```

在#renew方法中，不关注InstanceInfo，仅关注于租约本身以及租约的服务实例状态。如果根据服务实例的appName和instanceInfoId查询出服务实例的租约，并且根据#getOverriddenInstanceStatus方法得到的instanceStatus不为InstanceStatus.UNKNOWN，那么更新租约中的有效时间，即更新租约Lease中的lastUpdateTimestamp，达到续约的目的；如果租约不存在，那么返回续租失败的结果。

服务剔除 如果Eureka Client在注册后，既没有续约，也没有下线(服务崩溃或者网络异常等原因)，那么服务的状态就处于不可知的状态，不能保证能够从该服务实例中获取到回馈，所以需要服务剔除AbstractInstanceRegistry#evict方法定时清理这些不稳定的服务，该方法会批量将注册表中所有过期租约剔除。实现代码如下所示：

```java
// AbstractInstanceRegistry.java
@Override
public void evict() {
    evict(0l);
}
public void evict(long additionalLeaseMs) {
    // 自我保护相关，如果出现该状态，不允许剔除服务
    if (!isLeaseExpirationEnabled()) {
        return;
    }
    internalCancel(appName, id, false);
    // 遍历注册表register，一次性获取所有的过期租约
    List〈Lease〈InstanceInfo〉〉 expiredLeases = new ArrayList〈〉();
    for (Entry〈String, Map〈String, Lease〈InstanceInfo〉〉〉 groupEntry : registry.
        entrySet()) {
        Map〈String, Lease〈InstanceInfo〉〉 leaseMap = groupEntry.getValue();
        if (leaseMap != null) {
            for (Entry〈String, Lease〈InstanceInfo〉〉 leaseEntry : leaseMap.
                entrySet()) {
                Lease〈InstanceInfo〉 lease = leaseEntry.getValue();
                // 1
                if (lease.isExpired(additionalLeaseMs) &amp;&amp; lease.getHolder() != null) {
                    expiredLeases.add(lease);
                }
            }
        }
    }
    // 计算最大允许剔除的租约的数量，获取注册表租约总数
    int registrySize = (int) getLocalRegistrySize();
    // 计算注册表租约的阀值，与自我保护相关
    int registrySizeThreshold = (int) (registrySize * serverConfig.getRenewalPercentThreshold());
    int evictionLimit = registrySize - registrySizeThreshold;
    // 计算剔除租约的数量
    int toEvict = Math.min(expiredLeases.size(), evictionLimit);
    if (toEvict 〉 0) {
        Random random = new Random(System.currentTimeMillis());
        // 逐个随机剔除
        for (int i = 0; i 〈 toEvict; i++) {
            int next = i + random.nextInt(expiredLeases.size() - i);
            Collections.swap(expiredLeases, i, next);
            Lease〈InstanceInfo〉 lease = expiredLeases.get(i);
            String appName = lease.getHolder().getAppName();
            String id = lease.getHolder().getId();
            EXPIRED.increment();
            // 逐个剔除
            internalCancel(appName, id, false);
        }
    }
}

```

服务剔除将会遍历registry注册表，找出其中所有的过期租约，然后根据配置文件中续租百分比阀值和当前注册表的租约总数量计算出最大允许的剔除租约的数量(当前注册表中租约总数量减去当前注册表租约阀值)，分批次剔除过期的服务实例租约。对过期的服务实例租约调用AbstractInstanceRegistry#internalCancel服务下线的方法将其从注册表中清除掉。
服务剔除#evict方法中有很多限制，都是为了保证Eureka Server的可用性：

- 自我保护时期不能进行服务剔除操作。
- 过期操作是分批进行。
- 服务剔除是随机逐个剔除，剔除均匀分布在所有应用中，防止在同一时间内同一服务集群中的服务全部过期被剔除，以致大量剔除发生时，在未进行自我保护前促使了程序的崩溃。
服务剔除是一个定时的任务，所以AbstractInstanceRegistry中定义了一个EvictionTask用于定时执行服务剔除，默认为60秒一次。服务剔除的定时任务一般在AbstractInstanceRegistry初始化结束后进行，按照执行频率evictionIntervalTimerInMs的设定，定时剔除过期的服务实例租约。
自我保护机制主要在Eureka Client和Eureka Server之间存在网络分区的情况下发挥保护作用，在服务器端和客户端都有对应实现。假设在某种特定的情况下(如网络故障)，Eureka Client和Eureka Server无法进行通信，此时Eureka Client无法向Eureka Server发起注册和续约请求，Eureka Server中就可能因注册表中的服务实例租约出现大量过期而面临被剔除的危险，然而此时的Eureka Client可能是处于健康状态的(可接受服务访问)，如果直接将注册表中大量过期的服务实例租约剔除显然是不合理的。
针对这种情况，Eureka设计了“自我保护机制”。在Eureka Server处，如果出现大量的服务实例过期被剔除的现象，那么该Server节点将进入自我保护模式，保护注册表中的信息不再被剔除，在通信稳定后再退出该模式；在Eureka Client处，如果向Eureka Server注册失败，将快速超时并尝试与其他的Eureka Server进行通信。“自我保护机制”的设计大大提高了Eureka的可用性。

集群同步  

如果Eureka Server是通过集群的方式进行部署，那么为了维护整个集群中Eureka Server注册表数据的一致性，势必需要一个机制同步Eureka Server集群中的注册表数据。
Eureka Server集群同步包含两个部分，一部分是Eureka Server在启动过程中从它的peer节点中拉取注册表信息，并将这些服务实例的信息注册到本地注册表中；另一部分是Eureka Server每次对本地注册表进行操作时，同时会将操作同步到它的peer节点中，达到集群注册表数据统一的目的。

1.Eureka Server初始化本地注册表信息

```java
// PeerAwareInstanceRegistry.java
public int syncUp() {
    // 从临近的peer中复制整个注册表
    int count = 0;
    // 如果获取不到，线程等待
    count = 0;
    // 如果获取不到，线程等待
    for (int i = 0; ((i 〈 serverConfig.getRegistrySyncRetries()) &amp;&amp; (count == 0));
        i++) {
        if (i 〉 0) {
            try {
                Thread.sleep(serverConfig.getRegistrySyncRetryWaitMs());
            } catch (InterruptedException e) {
                break;
            }
        }
        // 获取所有的服务实例
        Applications apps = eurekaClient.getApplications();
        for (Application app : apps.getRegisteredApplications()) {
            for (InstanceInfo instance : app.getInstances()) {
                try {
                    // 判断是否可注册，主要用于AWS环境下进行，若部署在其他的环境，直接返回true
                    if (isRegisterable(instance)) {
                        // 注册到自身的注册表中
                        register(instance, instance.getLeaseInfo().getDurationInSecs(), true);
                        count++;
                    }
                } catch (Throwable t) {
                    logger.error("During DS init copy", t);
                }
            }
        }
    }
    return count;
}

```

Eureka Server也是一个Eureka Client，在启动的时候也会进行DiscoveryClient的初始化，会从其对应的Eureka Server中拉取全量的注册表信息。在Eureka Server集群部署的情况下，Eureka Server从它的peer节点中拉取到注册表信息后，将遍历这个Applications，将所有的服务实例通过AbstractRegistry#register方法注册到自身注册表中。
在初始化本地注册表时，Eureka Server并不会接受来自Eureka Client的通信请求(如注册、或者获取注册表信息等请求)。在同步注册表信息结束后会通过PeerAwareInstanceRegistryImpl#openForTraffic方法允许该Server接受流量。代码如下所示：

```java
    // PeerAwareInstanceRegistryImpl.java
public void openForTraffic(ApplicationInfoManager applicationInfoManager, int count) {
    // 初始化自我保护机制统计参数
    this.expectedNumberOfRenewsPerMin = count * 2;
    this.numberOfRenewsPerMinThreshold = (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
    this.startupTime = System.currentTimeMillis();
    // 如果同步的应用实例数量为0，将在一段时间内拒绝Client获取注册信息
    if (count 〉 0) {
        this.peerInstancesTransferEmptyOnStartup = false;
    }
    DataCenterInfo.Name selfName = applicationInfoManager.getInfo().getDataCenterInfo().getName();
    boolean isAws = Name.Amazon == selfName;
    // 判断是否是AWS运行环境，此处忽略
    if (isAws &amp;&amp; serverConfig.shouldPrimeAwsReplicaConnections()) {
        primeAwsReplicas(applicationInfoManager);
    }
    // 修改服务实例的状态为健康上线，可以接受流量
    applicationInfoManager.setInstanceStatus(InstanceStatus.UP);
    super.postInit();
}

```

在Eureka Server中有一个StatusFilter过滤器，用于检查Eureka Server的状态，当Server的状态不为UP时，将拒绝所有的请求。在Client请求获取注册表信息时，Server会判断此时是否允许获取注册表中的信息。上述做法是为了避免Eureka Server在#syncUp方法中没有获取到任何服务实例信息时(Eureka Server集群部署的情况下)，Eureka Server注册表中的信息影响到Eureka Client缓存的注册表中的信息。如果Eureka Server在#syncUp方法中没有获得任何的服务实例信息，它将把peerInstancesTransferEmptyOnStartup设置为true，这时该Eureka Server在WaitTimeInMsWhenSyncEmpty(可以通过eureka.server.wait-time-in-ms-when-sync-empty设置，默认是5分钟)时间后才能被Eureka Client访问获取注册表信息。

2.Eureka Server之间注册表信息的同步复制

为了保证Eureka Server集群运行时注册表信息的一致性，每个Eureka Server在对本地注册表进行管理操作时，会将相应的操作同步到所有peer节点中。
在PeerAwareInstanceRegistryImpl中，对Abstractinstanceregistry中的#register、#cancel和#renew等方法都添加了同步到peer节点的操作，使Server集群中注册表信息保持最终一致性，如下所示：

```java
//PeerAwareInstanceRegistryImpl.java
@Override
public boolean cancel(final String appName, final String id, final boolean isReplication) {
    if (super.cancel(appName, id, isReplication)) {
        // 同步下线状态
        replicateToPeers(Action.Cancel, appName, id, null, null, isReplication);
        replicateToPeers(Action.Cancel, appName, id, null, null, isReplication);
        ...
    }
    ...
}
public void register(final InstanceInfo info, final boolean isReplication) {
    int leaseDuration = Lease.DEFAULT_DURATION_IN_SECS;
    if (info.getLeaseInfo() != null &amp;&amp; info.getLeaseInfo().getDurationInSecs() 〉 0) {
            leaseDuration = info.getLeaseInfo().getDurationInSecs();
    }
    super.register(info, leaseDuration, isReplication);
    // 同步注册状态
    replicateToPeers(Action.Register, info.getAppName(), info.getId(), info, null, isReplication);
}
public boolean renew(final String appName, final String id, final boolean isReplication) {
    if (super.renew(appName, id, isReplication)) {
        // 同步续约状态
        replicateToPeers(Action.Heartbeat, appName, id, null, null, isReplication);
        return true;
    }
    return false;
}

//同步操作主要有
public enum Action {
    Heartbeat, Register, Cancel, StatusUpdate, DeleteStatusOverride;
    ...
}
//对此需要关注replicateToPeers方法，它将遍历Eureka Server中peer节点，向每个peer节点发送同步请求。代码如下所示：
//PeerAwareInstanceRegistryImpl.java
private void replicateToPeers(Action action, String appName, String id,
                              InstanceInfo info /* optional */,
                              InstanceStatus newStatus /* optional */, boolean isReplication) {
    Stopwatch tracer = action.getTimer().start();
    try {
        if (isReplication) {
            numberOfReplicationsLastMin.increment();
        }
        // 如果peer集群为空，或者这本来就是复制操作，那么就不再复制，防止造成循环复制
        if (peerEurekaNodes == Collections.EMPTY_LIST || isReplication) {
            return;
        }
        // 向peer集群中的每一个peer进行同步
        for (final PeerEurekaNode node : peerEurekaNodes.getPeerEurekaNodes()) {
            // 如果peer节点是自身的话，不进行同步复制
            if (peerEurekaNodes.isThisMyUrl(node.getServiceUrl())) {
                continue;
            }
            // 根据Action调用不同的同步请求
            replicateInstanceActionsToPeers(action, appName, id, info, newStatus, node);
        }
    } finally {
        tracer.stop();
    }
}
//PeerEurekaNode代表一个可同步共享数据的Eureka Server。在PeerEurekaNode中，具有register、cancel、heartbeat和statusUpdate等诸多用于向peer节点同步注册表信息的操作。
//PeerAwareInstanceRegistryImpl.java
private void replicateInstanceActionsToPeers(Action action, String appName,
    String id, InstanceInfo info, InstanceStatus newStatus, PeerEurekaNode node) {
    try {
        InstanceInfo infoFromRegistry = null;
        CurrentRequestVersion.set(Version.V2);
        switch (action) {
            case Cancel:
            // 同步下线
                node.cancel(appName, id);
                break;
            case Heartbeat:
                InstanceStatus overriddenStatus = overriddenInstanceStatusMap.get(id);
                infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                // 同步心跳
                node.heartbeat(appName, id, infoFromRegistry, overriddenStatus, false);
                break;
            case Register:
                // 同步注册
                node.register(info);
                break;
            case StatusUpdate:
                infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                // 同步状态更新
                node.statusUpdate(appName, id, newStatus, infoFromRegistry);
                break;
            case DeleteStatusOverride:
                infoFromRegistry = getInstanceByAppAndId(appName, id, false);
                node.deleteStatusOverride(appName, id, infoFromRegistry);
                break;
        }
    } catch (Throwable t) {
        logger.error("Cannot replicate information to {} for action {}", node.getServiceUrl(), action.name(), t);
}
```

PeerEurekaNode中的每一个同步复制都是通过批任务流的方式进行操作，同一时间段内相同服务实例的相同操作将使用相同的任务编号，在进行同步复制的时候根据任务编号合并操作，减少同步操作的数量和网络消耗，但是同时也造成同步复制的延时性，不满足CAP中的C(强一致性)。
通过Eureka Server在启动过程中初始化本地注册表信息和Eureka Server集群间的同步复制操作，最终达到了集群中Eureka Server注册表信息一致的目的。

获取注册表中服务实例信息()

Eureka Server中获取注册表的服务实例信息主要通过两个方法实现：AbstractInstanceRegistry#getApplicationsFromMultipleRegions从多地区获取全量注册表数据，AbstractInstanceRegistry#getApplicationDeltasFromMultipleRegions从多地区获取增量式注册表数据。

getApplicationsFromMultipleRegions方法将会从多个地区中获取全量注册表信息，并封装成Applications返回，实现代码如下所示：

```java
//AbstractInstanceRegistry.java
public Applications getApplicationsFromMultipleRegions(String[] remoteRegions) {
    boolean includeRemoteRegion = null != remoteRegions &amp;&amp; remoteRegions.length != 0;
    Applications apps = new Applications();
    apps.setVersion(1L);
    // 从本地registry获取所有的服务实例信息InstanceInfo
    for (Entry〈String, Map〈String, Lease〈InstanceInfo〉〉〉 entry : registry.entrySet()) {
        Application app = null;
        if (entry.getValue() != null) {
            for (Entry〈String, Lease〈InstanceInfo〉〉 stringLeaseEntry : entry.
                getValue().entrySet()) {
                Lease〈InstanceInfo〉 lease = stringLeaseEntry.getValue();
                if (app == null) {
                    app = new Application(lease.getHolder().getAppName());
                }
                app.addInstance(decorateInstanceInfo(lease));
            }
        }
        if (app != null) {
            apps.addApplication(app);
        }
    }
    if (includeRemoteRegion) {
        // 获取远程Region中的Eureka Server中的注册表信息
        ...
    }
    apps.setAppsHashCode(apps.getReconcileHashCode());
    return apps;
}
//它首先会将本地注册表registry中的所有服务实例信息提取出来封装到Applications中，再根据是否需要拉取远程Region中的注册表信息，将远程Region的Eureka Server注册表中的服务实例信息添加到Applications中。最后将封装了全量注册表信息的Applications返回给Client。
```

getApplicationDeltasFromMultipleRegions方法将会从多个地区中获取增量式注册表信息，并封装成Applications返回，实现代码如下所示：

```java
//AbstractInstanceRegistry.java
public Applications getApplicationDeltasFromMultipleRegions(String[] remoteRegions) {
    if (null == remoteRegions) {
        remoteRegions = allKnownRemoteRegions; // null means all remote regions.
    }
    boolean includeRemoteRegion = remoteRegions.length != 0;
    Applications apps = new Applications();
    apps.setVersion(responseCache.getVersionDeltaWithRegions().get());
    Map〈String, Application〉 applicationInstancesMap = new HashMap〈String, Application〉();
    try {
        write.lock();// 开启写锁
        // 遍历recentlyChangedQueue队列获取最近变化的服务实例信息InstanceInfo
        Iterator〈RecentlyChangedItem〉 iter = this.recentlyChangedQueue.iterator();
        while (iter.hasNext()) {
            //...
        }
        if (includeRemoteRegion) {
            // 获取远程Region中的Eureka Server的增量式注册表信息
           ...
        } finally {
        write.unlock();
    }
    // 计算应用集合一致性哈希码，用以在Eureka Client拉取时进行对比
    apps.setAppsHashCode(apps.getReconcileHashCode());
    return apps;
}

```

获取增量式注册表信息将会从recentlyChangedQueue中获取最近变化的服务实例信息。recentlyChangedQueue中统计了近3分钟内进行注册、修改和剔除的服务实例信息，在服务注册AbstractInstanceRegistry#registry、接受心跳请求AbstractInstanceRegistry#renew和服务下线AbstractInstanceRegistry#internalCancel等方法中均可见到recentlyChangedQueue对这些服务实例进行登记，用于记录增量式注册表信息。#getApplicationsFromMultipleRegions方法同样提供了从远程Region的Eureka Server获取增量式注册表信息的能力。

## Spring cloud OpenFeign





