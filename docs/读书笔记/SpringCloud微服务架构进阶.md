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

读者在阅读OpenFeign源码时，可以沿着两条路线进行，一是FeignServiceClient这样的被@FeignClient注解修饰的接口类(后续简称为FeignClient接口类)如何创建，也就是其Bean实例是如何被创建的；二是调用FeignServiceClient对象的网络请求相关的函数时，OpenFeign是如何发送网络请求的。而OpenFeign相关的类也可以以此来进行分类，一部分是用来初始化相应的Bean实例的，一部分是用来在调用方法时发送网络请求。

```java
@SpringBootApplication
@EnableFeignClients
public class FeignClientApplication {
    public static void main(String[] args) {
        SpringApplication.run(ChapterFeignClientApplication.class, args);
    }
}
```

@EnableFeignClients就像是一个开关，只有使用了该注解，OpenFeign相关的组件和配置机制才会生效。@EnableFeignClients还可以对OpenFeign相关组件进行自定义配置，

```java
@FeignClient("feign-service")
@RequestMapping("/feign-service")
public interface FeignServiceClient {
    @RequestMapping(value = "/instance/{serviceId}", method = RequestMethod.GET)
    public Instance getInstanceByServiceId(@PathVariable("serviceId") String serviceId);
}
```

### FeignClientsRegistrar

```java
//EnableFeignClients.java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
//ImportBeanDefinitionRegistrar的子类,用于处理@FeignClient注解
@Import(FeignClientsRegistrar.class)
public @interface EnableFeignClients {
    // 下面三个函数都是为了指定需要扫描的包
    String[] value() default {};
    String[] basePackages() default {};
    Class〈?〉[] basePackageClasses() default {};
    // 指定自定义feign client的自定义配置，可以配置Decoder、Encoder和Contract等组件,
    FeignClientsConfiguration是默认的配置类
    Class〈?〉[] defaultConfiguration() default {};
    // 指定被@FeignClient修饰的类，如果不为空，那么路径自动检测机制会被关闭
    Class〈?〉[] clients() default {};
}
```

上面的代码中，FeignClientsRegistrar是ImportBeanDefinitionRegistrar的子类，Spring用ImportBeanDefinitionRegistrar来动态注册BeanDefinition。OpenFeign通过FeignClientsRegistrar来处理@FeignClient修饰的FeignClient接口类，将这些接口类的BeanDefinition注册到Spring容器中，这样就可以使用@Autowired等方式来自动装载这些FeignClient接口类的Bean实例。FeignClientsRegistrar的部分代码如下所示：

```java
//FeignClientsRegistrar.java
class FeignClientsRegistrar implements ImportBeanDefinitionRegistrar,
        ResourceLoaderAware, BeanClassLoaderAware, EnvironmentAware {
    ...
    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata,
        BeanDefinitionRegistry registry) {
        //从EnableFeignClients的属性值来构建Feign的自定义Configuration进行注册
        registerDefaultConfiguration(metadata, registry);
        //扫描package，注册被@FeignClient修饰的接口类的Bean信息
        registerFeignClients(metadata, registry);
    }
    ...
}
```

如上述代码所示，FeignClientsRegistrar的registerBeanDefinitions方法主要做了两个事情:
一是注册@EnableFeignClients提供的自定义配置类中的相关Bean实例
二是根据@EnableFeignClients提供的包信息扫描@FeignClient注解修饰的FeignCleint接口类，然后进行Bean实例注册
@EnableFeignClients的自定义配置类是被@Configuration注解修饰的配置类，它会提供一系列组装FeignClient的各类组件实例。这些组件包括：Client、Targeter、Decoder、Encoder和Contract等。接下来看看registerDefaultConfiguration的代码实现，如下所示：

```java
   //FeignClientsRegistrar.java
private void registerDefaultConfiguration(AnnotationMetadata metadata,
            BeanDefinitionRegistry registry) {
    //获取到metadata中关于EnableFeignClients的属性值键值对
    Map〈String, Object〉 defaultAttrs = metadata
            .getAnnotationAttributes(EnableFeignClients.class.getName(), true);
     // 如果EnableFeignClients配置了defaultConfiguration类，那么才进行下一步操作，如果没有，会使用默认的FeignConfiguration
    if (defaultAttrs != null && defaultAttrs.containsKey("defaultConfiguration")) {
        String name;
        if (metadata.hasEnclosingClass()) {
            name = "default." + metadata.getEnclosingClassName();
        }
        else {
            name = "default." + metadata.getClassName();
        }
        registerClientConfiguration(registry, name,
                defaultAttrs.get("defaultConfiguration"));
    }
  }
```

如上述代码所示，registerDefaultConfiguration方法会判断@EnableFeignClients注解是否设置了defaultConfiguration属性。如果有，则将调用registerClientConfiguration方法，进行BeanDefinitionRegistry的注册。registerClientConfiguration方法的代码如下所示。

```java
/ FeignClientsRegistrar.java
private void registerClientConfiguration(BeanDefinitionRegistry registry, Object name,
    Object configuration) {
    // 使用BeanDefinitionBuilder来生成BeanDefinition，并注册到registry上
    BeanDefinitionBuilder builder = BeanDefinitionBuilder
        .genericBeanDefinition(FeignClientSpecification.class);
    builder.addConstructorArgValue(name);
    builder.addConstructorArgValue(configuration);
    registry.registerBeanDefinition(
        name + "." + FeignClientSpecification.class.getSimpleName(),
        builder.getBeanDefinition());
}
```

BeanDefinitionRegistry是Spring框架中用于动态注册BeanDefinition信息的接口，调用其registerBeanDefinition方法可以将BeanDefinition注册到Spring容器中，其中name属性就是注册BeanDefinition的名称。

FeignClientSpecification类实现了NamedContextFactory.Specification接口，它是OpenFeign组件实例化的重要一环，它持有自定义配置类提供的组件实例，供OpenFeign使用。Spring Cloud框架使用NamedContextFactory创建一系列的运行上下文(ApplicationContext)，来让对应的Specification在这些上下文中创建实例对象。这样使得各个子上下文中的实例对象相互独立，互不影响，可以方便地通过子上下文管理一系列不同的实例对象。NamedContextFactory有三个功能，一是创建AnnotationConfigApplicationContext子上下文；二是在子上下文中创建并获取Bean实例；三是当子上下文消亡时清除其中的Bean实例。在OpenFeign中，FeignContext继承了NamedContextFactory，用于存储各类OpenFeign的组件实例。

FeignAutoConfiguration是OpenFeign的自动配置类，它会提供FeignContext实例。并且将之前注册的FeignClientSpecification通过setConfigurations方法设置给FeignContext实例。这里处理了默认配置类FeignClientsConfiguration和自定义配置类的替换问题。果FeignClientsRegistrar没有注册自定义配置类，那么configurations将不包含FeignClientSpecification对象，否则会在setConfigurations方法中进行默认配置类的替换。

```java
//FeignAutoConfiguration.java
@Autowired(required = false)
private List〈FeignClientSpecification〉 configurations = new ArrayList〈〉();
@Bean
public FeignContext feignContext() {
    FeignContext context = new FeignContext();
    context.setConfigurations(this.configurations);
    return context;
}
//FeignContext.java
public class FeignContext extends NamedContextFactory〈FeignClientSpecification〉 {
    public FeignContext() {
        //将默认的FeignClientConfiguration作为参数传递给构造函数
        super(FeignClientsConfiguration.class, "feign", "feign.client.name");
    }
}
```

.扫描类信息
FeignClientsRegistrar做的第二件事情是扫描指定包下的类文件，注册@FeignClient注解修饰的接口类信息，如下所示：

```java
//FeignClientsRegistrar.java
public void registerFeignClients(AnnotationMetadata metadata,
        BeanDefinitionRegistry registry) {
    //生成自定义的ClassPathScanningProvider
    ClassPathScanningCandidateComponentProvider scanner = getScanner();
    scanner.setResourceLoader(this.resourceLoader);
    Set〈String〉 basePackages;
    //获取EnableFeignClients所有属性的键值对
    Map〈String, Object〉 attrs = metadata
            .getAnnotationAttributes(EnableFeignClients.class.getName());
    //依照Annotation来进行TypeFilter，只会扫描出被FeignClient修饰的类
    AnnotationTypeFilter annotationTypeFilter = new AnnotationTypeFilter(
            FeignClient.class);
    final Class〈?〉[] clients = attrs == null ? null
            : (Class〈?〉[]) attrs.get("clients");
    //如果没有设置clients属性，那么需要扫描basePackage，所以设置了AnnotationTypeFilter,
           并且去获取basePackage
    if (clients == null || clients.length == 0) {
        scanner.addIncludeFilter(annotationTypeFilter);
        basePackages = getBasePackages(metadata);
    }
    //代码有删减，遍历上述过程中获取的basePackages列表
    for (String basePackage : basePackages) {
        //获取basepackage下的所有BeanDefinition
        Set〈BeanDefinition〉 candidateComponents = scanner
                .findCandidateComponents(basePackage);
        for (BeanDefinition candidateComponent : candidateComponents) {
            if (candidateComponent instanceof AnnotatedBeanDefinition) {
                AnnotatedBeanDefinition beanDefinition = (AnnotatedBeanDefinition) candidateComponent;
                AnnotationMetadata annotationMetadata = beanDefinition.getMetadata();
                //从这些BeanDefinition中获取FeignClient的属性值
                Map〈String, Object〉 attributes = annotationMetadata
                        .getAnnotationAttributes(
                                FeignClient.class.getCanonicalName());
                String name = getClientName(attributes);
                //对单独某个FeignClient的configuration进行配置
                registerClientConfiguration(registry, name,
                        attributes.get("configuration"));
                //注册FeignClient的BeanDefinition
                registerFeignClient(registry, annotationMetadata, attributes);
            }
        }
    }
}
```

如上述代码所示，FeignClientsRegistrar的registerFeignClients方法依据@EnableFeignClients的属性获取要扫描的包路径信息，然后获取这些包下所有被@FeignClient注解修饰的接口类的BeanDefinition，最后调用registerFeignClient动态注册BeanDefinition。registerFeignClients方法中有一些细节值得认真学习，有利于加深了解Spring框架。首先是如何自定义Spring类扫描器，即如何使用ClassPathScanningCandidateComponentProvider和各类TypeFilter。
OpenFeign使用了AnnotationTypeFilter，来过滤出被@FeignClient修饰的类，getScanner方法的具体实现如下所示：

```java
//FeignClientsRegistrar.java
protected ClassPathScanningCandidateComponentProvider getScanner() {
    return new ClassPathScanningCandidateComponentProvider(false, this.environment) {
        @Override
        protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
            boolean isCandidate = false;
            //判断beanDefinition是否为内部类，否则直接返回false
            if (beanDefinition.getMetadata().isIndependent()) {
                //判断是否为接口类，所实现的接口只有一个，并且该接口是Annotation。否则直接
                       返回true
                if (!beanDefinition.getMetadata().isAnnotation()) {
                    isCandidate = true;
                }
            }
            return isCandidate;
        }
    };
}
```

ClassPathScanningCandidateComponentProvider的作用是遍历指定路径的包下的所有类。比如指定包路径为com/test/openfeign，它会找出com.test.openfeign包下所有的类，将所有的类封装成Resource接口集合。Resource接口是Spring对资源的封装，有FileSystemResource、ClassPathResource、UrlResource等多种实现。接着ClassPathScanningCandidateComponentProvider类会遍历Resource集合，通过includeFilters和excludeFilters两种过滤器进行过滤操作。includeFilters和excludeFilters是TypeFilter接口类型实例的集合，TypeFilter接口是一个用于判断类型是否满足要求的类型过滤器。excludeFilters中只要有一个TypeFilter满足条件，这个Resource就会被过滤掉；而includeFilters中只要有一个TypeFilter满足条件，这个Resource就不会被过滤。如果一个Resource没有被过滤，它会被转换成ScannedGenericBeanDefinition添加到BeanDefinition集合中。

FeignClientFactoryBean是工厂类，Spring容器通过调用它的getObject方法来获取对应的Bean实例。被@FeignClient修饰的接口类都是通过FeignClientFactoryBean的getObject方法来进行实例化的，具体实现如下代码所示：

```java
FeignClientFactoryBean是工厂类，Spring容器通过调用它的getObject方法来获取对应的Bean实例。被@FeignClient修饰的接口类都是通过FeignClientFactoryBean的getObject方法来进行实例化的，具体实现如下代码所示：
//FeignClientFactoryBean.java
public Object getObject() throws Exception {
    FeignContext context = applicationContext.getBean(FeignContext.class);
    Feign.Builder builder = feign(context);
    if (StringUtils.hasText(this.url) &amp;&amp; !this.url.startsWith("http")) {
        this.url = "http://" + this.url;
    }
    String url = this.url + cleanPath();
    //调用FeignContext的getInstance方法获取Client对象
    Client client = getOptional(context, Client.class);
    //因为有具体的Url,所以就不需要负载均衡，所以除去LoadBalancerFeignClient实例
    if (client != null) {
        if (client instanceof LoadBalancerFeignClient) {
            client = ((LoadBalancerFeignClient)client).getDelegate();
        }
        builder.client(client);
    }
    Targeter targeter = get(context, Targeter.class);
    return targeter.target(this, builder, context, new HardCodedTarget〈〉(
            this.type, this.name, url));
}

//这里就用到了FeignContext的getInstance方法，我们在前边已经讲解了FeignContext的作用，getOptional方法调用了FeignContext的getInstance方法，从FeignContext的对应名称的子上下文中获取到Client类型的Bean实例，其具体实现如下所示：
//NamedContextFactory.java
public 〈T〉 T getInstance(String name, Class〈T〉 type) {
    AnnotationConfigApplicationContext context = getContext(name);
    if (BeanFactoryUtils.beanNamesForTypeIncludingAncestors(context,
            type).length 〉 0) {
        //从对应的context中获取Bean实例,如果对应的子上下文没有则直接从父上下文中获取
        return context.getBean(type);
    }
    return null;
}

//Targeter是一个接口，它的target方法会生成对应的实例对象。它有两个实现类，分别为DefaultTargeter和HystrixTargeter。OpenFeign使用HystrixTargeter这一层抽象来封装关于Hystrix的实现。DefaultTargeter的实现如下所示，只是调用了Feign.Builder的target方法：
//DefaultTargeter.java
class DefaultTargeter implements Targeter {
@Override
    public 〈T〉 T target(FeignClientFactoryBean factory, Feign.Builder feign, FeignContext context,
                        Target.HardCodedTarget〈T〉 target) {
        return feign.target(target);
    }
}

//而Feign.Builder是由FeignClientFactoryBean对象的feign方法创建的。Feign.Builder会设置FeignLoggerFactory、EncoderDecoder和Contract等组件，这些组件的Bean实例都是通过FeignContext获取的，也就是说这些实例都是可配置的，你可以通过OpenFeign的配置机制为不同的FeignClient配置不同的组件实例。feign方法的实现如下所示：
//FeignClientFactoryBean.java
protected Feign.Builder feign(FeignContext context) {
    FeignLoggerFactory loggerFactory = get(context, FeignLoggerFactory.class);
    Logger logger = loggerFactory.create(this.type);
    Feign.Builder builder = get(context, Feign.Builder.class)
        .logger(logger)
        .encoder(get(context, Encoder.class))
        .decoder(get(context, Decoder.class))
        .contract(get(context, Contract.class));
    configureFeign(context, builder);
    return builder;
}
//Feign.Builder负责生成被@FeignClient修饰的FeignClient接口类实例。它通过Java反射机制，构造InvocationHandler实例并将其注册到FeignClient上，当FeignClient的方法被调用时，InvocationHandler的回调函数会被调用，OpenFeign会在其回调函数中发送网络请求。build方法如下所示：
//Feign.Builder
public Feign build() {
    SynchronousMethodHandler.Factory synchronousMethodHandlerFactory =
        new SynchronousMethodHandler.Factory(client, retryer, requestInterceptors,
        logger, logLevel, decode404);
    ParseHandlersByName handlersByName = new ParseHandlersByName(contract, options, encoder, decoder, errorDecoder, synchronousMethodHandlerFactory);
    return new ReflectiveFeign(handlersByName, invocationHandlerFactory);
}

//ReflectiveFeign的newInstance方法是生成FeignClient实例的关键实现。它主要做了两件事情，一是扫描FeignClient接口类的所有函数，生成对应的Handler；二是使用Proxy生成FeignClient的实例对象，代码如下所示：

//ReflectiveFeign.java
public 〈T〉 T newInstance(Target〈T〉 target) {
    Map〈String, MethodHandler〉 nameToHandler = targetToHandlersByName.apply(target);
    Map〈Method, MethodHandler〉 methodToHandler = new LinkedHashMap〈Method, MethodHandler〉();
    List〈DefaultMethodHandler〉 defaultMethodHandlers = new LinkedList〈DefaultMethodHandler〉();
    for (Method method : target.type().getMethods()) {
        if (method.getDeclaringClass() == Object.class) {
            continue;
        } else if(Util.isDefault(method)) {
            //为每个默认方法生成一个DefaultMethodHandler
            defaultMethodHandler handler = new DefaultMethodHandler(method);
            defaultMethodHandlers.add(handler);
            methodToHandler.put(method, handler);
        } else {
            methodToHandler.put(method, nameToHandler.get(Feign.configKey(target.type(), method)));
        }
    }
    //生成java reflective的InvocationHandler
    InvocationHandler handler = factory.create(target, methodToHandler);
    T proxy = (T) Proxy.newProxyInstance(target.type().getClassLoader(), new Class〈?〉[] {target.type()}, handler);
    //将defaultMethodHandler绑定到proxy中
    for(DefaultMethodHandler defaultMethodHandler : defaultMethodHandlers) {
        defaultMethodHandler.bindTo(proxy);
    }
    return proxy;
}

```

1.扫描函数信息
在扫描FeignClient接口类所有函数生成对应Handler的过程中，OpenFeign会生成调用该函数时发送网络请求的模板，也就是RequestTemplate实例。RequestTemplate中包含了发送网络请求的URL和函数参数填充的信息。@RequestMapping、@PathVariable等注解信息也会包含到RequestTemplate中，用于函数参数的填充。ParseHandlersByName类的apply方法就是这一过程的具体实现。它首先会使用Contract来解析接口类中的函数信息，并检查函数的合法性，然后根据函数的不同类型来为每个函数生成一个BuildTemplateByResolvingArgs对象，最后使用SynchronousMethodHandler.Factory来创建MethodHandler实例。ParseHandlersByName的apply实现如下代码所示：

```java
ParseHandlersByName.java
public Map〈String, MethodHandler〉 apply(Target key) {
    // 获取type的所有方法的信息,会根据注解生成每个方法的RequestTemplate
    List〈MethodMetadata〉 metadata = contract.parseAndValidatateMetadata(key.type());
    Map〈String, MethodHandler〉 result = new LinkedHashMap〈String, MethodHandler〉();
    for (MethodMetadata md : metadata) {
    BuildTemplateByResolvingArgs buildTemplate;
    if (!md.formParams().isEmpty() &amp;&amp; md.template().bodyTemplate() == null) {
        buildTemplate = new BuildFormEncodedTemplateFromArgs(md, encoder);
    } else if (md.bodyIndex() != null) {
        buildTemplate = new BuildEncodedTemplateFromArgs(md, encoder);
    } else {
        buildTemplate = new BuildTemplateByResolvingArgs(md);
    }
    result.put(md.configKey(),
        factory.create(key, md, buildTemplate, options, decoder, errorDecoder));
    }
    return result;
}
```

### 函数调用和网络请求

在配置和实例生成结束之后，就可以直接使用FeignClient接口类的实例，调用它的函数来发送网络请求。在调用其函数的过程中，由于设置了MethodHandler，所以最终函数调用会执行SynchronousMethodHandler的invoke方法。在该方法中，OpenFeign会将函数的实际参数值与之前生成的RequestTemplate进行结合，然后发送网络请求。

```java
//SynchronousMethodHandler.java
final class SynchronousMethodHandler implements MethodHandler {
    public Object invoke(Object[] argv) throws Throwable {
        //根据函数参数创建RequestTemplate实例，buildTemplateFromArgs是RequestTemplate.
          Factory接口的实例，在当前状况下是
        //BuildTemplateByResolvingArgs类的实例
        RequestTemplate template = buildTemplateFromArgs.create(argv);
        Retryer retryer = this.retryer.clone();
        while (true) {
            try {
                return executeAndDecode(template);
            } catch (RetryableException e) {
                retryer.continueOrPropagate(e);
                if (logLevel != Logger.Level.NONE) {
                    logger.logRetry(metadata.configKey(), logLevel);
                }
                continue;
            }
        }
    }
}
```

executeAndDecode方法会根据RequestTemplate生成Request对象，然后交给Client实例发送网络请求，最后返回对应的函数返回类型的实例。executeAndDecode方法的具体实现如下所示：

```java
//SynchronousMethodHandler.java
Object executeAndDecode(RequestTemplate template) throws Throwable {
    //根据RequestTemplate生成Request
    Request request = targetRequest(template);
    Response response;
    //client发送网络请求，client可能为okhttpclient和apacheClient
    try {
        response = client.execute(request, options);
        response.toBuilder().request(request).build();
    } catch (IOException e) {    } catch (IOException e) {
        //...
    }
    try {
        //如果response的类型就是函数返回类型，那么可以直接返回
        if (Response.class == metadata.returnType()) {
            if (response.body() == null) {
                return response;
            }
            // 设置body
            byte[] bodyData = Util.toByteArray(response.body().asInputStream());
            return response.toBuilder().body(bodyData).build();
          }
        } catch (IOException e) {
            //...
    }
}
```

Client是用来发送网络请求的接口类，有OkHttpClient和RibbonClient两个子类。OkhttpClient调用OkHttp的相关组件进行网络请求的发送。OkHttpClient的具体实现如下所示：

```java
/OkHttpClient.java
public feign.Response execute(feign.Request input, feign.Request.Options options)
        throws IOException {
    //将feign.Request转换为Oktthp的Request对象
       //将feign.Request转换为Oktthp的Request对象
    Request request = toOkHttpRequest(input);
    //使用Okhttp的同步操作发送网络请求
    Response response = requestOkHttpClient.newCall(request).execute();
    //将Okhttp的Response转换为feign.Response
    return toFeignResponse(response).toBuilder().request(input).build();
}
```

## 断路器Hystrix

在分布式系统中，不同服务之间的调用非常常见，当服务提供者不可用时就很有可能发生服务雪崩效应，导致整个系统的不可用。所以为了预防这种情况的发生，可以使用断路器模式进行预防(类比电路中的断路器，在电路过大的时候自动断开，防止电线过热损害整条电路)。
断路器将远程方法调用包装到一个断路器对象中，用于监控方法调用过程的失败。一旦该方法调用发生的失败次数在一段时间内达到一定的阀值，那么这个断路器将会跳闸，在接下来时间里再次调用该方法将会被断路器直接返回异常，而不再发生该方法的真实调用。这样就避免了服务调用者在服务提供者不可用时发送请求，从而减少线程池中资源的消耗，保护了服务调用者。图6-2为断路器时序图。
如图6-2所示，虽然断路器在打开的时候避免了被保护方法的无效调用，但是当情况恢复正常时，需要外部干预来重置断路器，使得方法调用可以重新发生。所以合理的断路器应该具备一定的开关转化逻辑，它需要一个机制来控制它的重新闭合，图6-3展示了一个通过重置时间来决定断路器的重新闭合的逻辑。

- 关闭状态：断路器处于关闭状态，统计调用失败次数，在一段时间内达到一定的阀值后断路器打开。
- 打开状态：断路器处于打开状态，对方法调用直接返回失败错误，不发生真正的方法调用。设置了一个重置时间，在重置时间结束后，断路器来到半开状态。
- 半开状态：断路器处于半开状态，此时允许进行方法调用，当调用都成功了(或者成功到达一定的比例)，关闭断路器，否则认为服务没有恢复，重新打开断路器。
断路器的打开能保证服务调用者在调用异常服务时，快速返回结果，避免大量的同步等待，减少服务调用者的资源消耗。并且断路器能在打开一段时间后继续侦测请求执行结果，判断断路器是否能关闭，恢复服务的正常调用。

### 服务降级操作

在Hystrix中，当服务间调用发生问题时，它将采用备用的Fallback方法代替主方法执行并返回结果，对失败服务进行了服务降级。当调用服务失败次数在一段时间内超过了断路器的阀值时，断路器将打开，不再进行真正的方法调用，而是快速失败，直接执行Fallback逻辑，服务降级，减少服务调用者的资源消耗，保护服务调用者中的线程资源。

### 资源隔离

在货船中，为了防止漏水和火灾的扩散，一般会将货仓进行分割，避免了一个货仓出事导致整艘船沉没的悲剧。同样的，在Hystrix中，也采用了舱壁模式，将系统中的服务提供者隔离起来，一个服务提供者延迟升高或者失败，并不会导致整个系统的失败，同时也能够控制调用这些服务的并发度。
1.线程与线程池
Hystrix通过将调用服务线程与服务访问的执行线程分隔开来，调用线程能够空出来去做其他的工作而不至于因为服务调用的执行阻塞过长时间。
在Hystrix中，将使用独立的线程池对应每一个服务提供者，用于隔离和限制这些服务。于是，某个服务提供者的高延迟或者饱和资源受限只会发生在该服务提供者对应的线程池中。
如图6-5所示，Dependency D的调用失败或者高延迟仅会导致自身对应的线程池中的5个线程阻塞，并不会影响其他服务提供者的线程池。系统完全与服务提供者请求隔离开来，即使服务提供者对应的线程完全耗尽，并不会影响系统中的其他请求。注意在服务提供者的线程池被占满时，对该服务提供者的调用会被Hystrix直接进入回滚逻辑，快速失败，保护服务调用者的资源稳定。

除了线程池外，Hystrix还可以通过信号量(计数器)来限制单个服务提供者的并发量。如果通过信号量来控制系统负载，将不再允许设置超时控制和异步化调用，这就表示在服务提供者出现高延迟时，其调用线程将会被阻塞，直至服务提供者的网络请求超时。如果对服务提供者的稳定性有足够的信心，可以通过信号量来控制系统的负载。

### 设计思路

结合上面的介绍，我们可以简单理解一下Hystrix的实现思路：
·它将所有的远程调用逻辑封装到HystrixCommand或者HystrixObservableCommand对象中，这些远程调用将会在独立的线程中执行(资源隔离)，这里使用了设计模式中的命令模式。
·Hystrix对访问耗时超过设置阀值的请求采用自动超时的策略。该策略对所有的命令都有效(如果资源隔离的方式为信号量，该特性将失效)，超时的阀值可以通过命令配置进行自定义。
·为每一个服务提供者维护一个线程池(或者信号量)，当线程池被占满时，对于该服务提供者的请求将会被直接拒绝(快速失败)而不是排队等待，减少系统的资源等待。
·针对请求服务提供者划分出成功、失效、超时和线程池被占满等四种可能出现的情况。
·断路器机制将在请求服务提供者失败次数超过一定阀值后手动或者自动切断服务一段时间。
·当请求服务提供者出现服务拒绝、超时和短路(多个服务提供者依次顺序请求，前面的服务提供者请求失败，后面的请求将不会发出)等情况时，执行其Fallback方法，服务降级。
·提供接近实时的监控和配置变更服务。

### 源码解析

简单的流程如下：
1)构建HystrixCommand或者HystrixObservableCommand对象。
2)执行命令。
3)检查是否有相同命令执行的缓存。
4)检查断路器是否打开。
5)检查线程池或者信号量是否被消耗完。
6)调用HystrixObservableCommand#construct或HystrixCommand#run执行被封装的远程调用逻辑。
7)计算链路的健康情况。
8)在命令执行失败时获取Fallback逻辑。
9)返回成功的Observable。
接着我们通过源码来逐步理解这些过程。

HystrixCommand可以继承 自定义 默认资源隔离10线程 10信号量 50失败熔断

### 合并请求

Hystrix还提供了请求合并的功能。多个请求被合并为一个请求进行一次性处理，可以有效减少网络通信和线程池资源。请求合并之后，一个请求原本可能在6毫秒之内能够结束，现在必须等待请求合并周期后(10毫秒)才能发送请求，增加了请求的时间(16毫秒)。但是请求合并在处理高并发和高延迟命令上效果极佳。

它提供两种方式进行请求合并：request-scoped收集一个HystrixRequestContext中的请求集合成一个批次；而globally-scoped将多个HystrixRequestContext中的请求集合成一个批次，这需要应用的下游依赖能够支持在一个命令调用中处理多个HystrixRequestContext。
HystrixRequestContext中包含和管理着HystrixRequestVariableDefault，HystrixRequestVariableDefault中提供了请求范围内的相关变量，所以在同一请求中的多个线程可以分享状态，HystrixRequestContext也可以通过HystrixRequestVariableDefault收集到请求范围内相同的HystrixCommand进行合并。

.通过注解方式进行请求合并
单个请求需要使用@HystrixCollapser注解修饰，并指明batchMethod方法，这里我们设置请求合并的周期为100秒。由于请求合并中不能同步等待结果，所以单个请求返回的结果为Future，即需要异步等待结果。
batchMethod方法需要被@HystrixCommand注解，说明这是一个被HystrixCommand封装的方法，其内是一个批量的请求接口，为了方便展示，例中就直接虚假地构建了本地数据，同时有日志打印批量方法被执行。具体代码如下所示

```java
@HystrixCollapser(batchMethod = "getInstanceByServiceIds",
    collapserProperties = {@HystrixProperty(name ="timerDelayInMilliseconds",value = "100")})
public Future〈Instance〉 getInstanceByServiceId(String serviceId) {
    return null;
}
@HystrixCommand
public List〈Instance〉 getInstanceByServiceIds(List〈String〉 serviceIds){
    List〈Instance〉 instances = new ArrayList〈〉();
    logger.info("start batch!");
    for(String s : serviceIds){
        instances.add(new Instance(s, DEFAULT_HOST, DEFAULT_PORT));
    }
    return instances;
}
```

2.继承HystrixCollapser
请求合并命令同样也可以通过自定义的方式实现，只需继承HystrixCollapser抽象类，如下所示：

```java
public class CustomCollapseCommand extends HystrixCollapser〈List〈Instance〉, Instance, String〉{
    public String serviceId;
    public CustomCollapseCommand(String serviceId){
        super(Setter.withCollapserKey(HystrixCollapserKey.Factory.asKey("customCollapseCommand")));
        this.serviceId = serviceId;
    }
    @Override
    public String getRequestArgument() {
        return serviceId;
    }
    @Override
    protected HystrixCommand〈List〈Instance〉〉 createCommand(Collection〈CollapsedRequest〈Instance, String〉〉 collapsedRequests) {
        List〈String〉 ids = collapsedRequests.stream().map(CollapsedRequest::getArgument).collect(Collectors.toList());
        return new InstanceBatchCommand(ids);
    }
    @Override
    protected void mapResponseToRequests(List〈Instance〉 batchResponse, Collection〈CollapsedRequest〈Instance, String〉〉 collapsedRequests) {
        int count = 0;
        for( CollapsedRequest〈Instance, String〉 request : collapsedRequests){
            request.setResponse(batchResponse.get(count++));
        }
    }
    private static final class InstanceBatchCommand extends HystrixCommand〈List〈Instance〉〉{
        private List〈String〉 serviceIds;
        private static String DEFAULT_SERVICE_ID = "application";
        private static String DEFAULT_HOST = "localhost";
        private static int DEFAULT_PORT = 8080;
        private static Logger logger = LoggerFactory.getLogger(InstanceBatchCommand.class);
        protected InstanceBatchCommand(List〈String〉 serviceIds) {
            super(HystrixCommandGroupKey.Factory.asKey("instanceBatchGroup"));
            this.serviceIds = serviceIds;
        }
        @Override
        protected List〈Instance〉 run() throws Exception {
            List〈Instance〉 instances = new ArrayList〈〉();
            logger.info("start batch!");
            for(String s : serviceIds){
                instances.add(new Instance(s, DEFAULT_HOST, DEFAULT_PORT));
            }
            return instances;
        }
    }
}

继承HystrixCollapser需要指定三个泛型，如下所示：
public abstract class HystrixCollapser〈BatchReturnType, ResponseType, RequestArgumentType〉 implements HystrixExecutable〈ResponseType〉, HystrixObservable〈ResponseType〉 {...}

BatchReturnType是批量操作的返回值类型，例子中为List〈Instance〉；ResponseType是单个操作的返回值类型，例子中是Instance；RequestArgumentType是单个操作的请求参数类型，例子中是String类型。
在构造函数中需要指定CollapserKey，用来标记被合并请求的键值。CustomCollapseCommand同样可以通过Setter的方式修改默认配置。
同时还需要实现三个方法，getRequestArgument方法获取被合并的单个请求的参数，createCommand方法用来生成进行批量请求的命令Command，mapResponseToRequests方法是将批量请求的结果与合并的请求进行匹配以返回对应的请求结果。

```

## Ribbon

Ribbon是管理HTTP和TCP服务客户端的负载均衡器。Ribbon具有一系列带有名称的客户端(Named Client)，也就是带有名称的Ribbon客户端(Ribbon Client)。每个客户端由可配置的组件构成，负责一类服务的调用请求。Spring Cloud通过RibbonClientConfiguration为每个Ribbon客户端创建一个ApplicationContext上下文来进行组件装配。Ribbon作为Spring Cloud的负载均衡机制的实现，可以与OpenFeign和RestTemplate进行无缝对接，让二者具有负载均衡的能力。

当系统面临大量的用户访问，负载过高的时候，通常会增加服务器数量来进行横向扩展，多个服务器的负载需要均衡，以免出现服务器负载不均衡，部分服务器负载较大，部分服务器负载较小的情况。通过负载均衡，使得集群中服务器的负载保持在稳定高效的状态，从而提高整个系统的处理能力。
系统的负载均衡分为软件负载均衡和硬件负载均衡。软件负载均衡使用独立的负载均衡程序或系统自带的负载均衡模块完成对请求的分配派发。硬件负载均衡通过特殊的硬件设备进行负载均衡的调配。
而软负载均衡一般分为两种类型，基于DNS的负载均衡和基于IP的负载均衡。利用DNS实现负载均衡，就是在DNS服务器配置多个A记录，不同的DNS请求会解析到不同的IP地址。大型网站一般使用DNS作为第一级负载均衡。基于IP的负载均衡根据请求的IP进行负载均衡。LVS就是具有代表性的基于IP的负载均衡实现。
但是目前来说，大家最为熟悉的，最为主流的负载均衡技术还是反向代理负载均衡。所有主流的Web服务器都热衷于支持基于反向代理的负载均衡。它的核心工作是代理根据一定规则，将HTTP请求转发到服务器集群的单一服务器上。
Ribbon使用的是客户端负载均衡。客户端负载均衡和服务端负载均衡最大的区别在于服务端地址列表的存储位置，在客户端负载均衡中，所有的客户端节点都有一份自己要访问的服务端地址列表，这些列表统统都是从服务注册中心获取的；而在服务端负载均衡中，客户端节点只知道单一服务代理的地址，服务代理则知道所有服务端的地址。在Spring Cloud中我们如果想要使用客户端负载均衡，方法很简单，使用@LoadBalanced注解即可，这样客户端在发起请求的时候会根据负载均衡策略从服务端列表中选择一个服务端，向该服务端发起网络请求，从而实现负载均衡。

### 配置和实例初始化

1111

### 与openfeign的集成

Ribbon是RESTful HTTP客户端OpenFeign负载均衡的默认实现。本书在OpenFeign的章节中讲解了相关实例的初始化过程。FeignClientFactoryBean是创造FeignClient的工厂类，在其getObject方法中有一个分支判断，当请求URL不为空时，就会生成一个具有负载均衡的FeignClient。在这个过程中，OpenFeign就默认引入了Ribbon的负载均衡实现，OpenFegin引入Ribbon的部分代码如下所示：

```java
//FeignClientFactoryBean.java
public Object getObject() throws Exception {
    FeignContext context = applicationContext.getBean(FeignContext.class);
    Feign.Builder builder = feign(context);
    //如果url不为空，则需要负载均衡
    if (!StringUtils.hasText(this.url)) {
        String url;
        if (!this.name.startsWith("http")) {
            url = "http://" + this.name;
        }
        else {
            url = this.name;
        }
        url += cleanPath();
        return loadBalance(builder, context, new HardCodedTarget〈〉(this.type,
                this.name, url));
    }
    //....生成普通的FeignClient
}
```

如OpenFeign的源码所示，loadBalance方法会生成LoadBalancerFeignClient实例进行返回。LoadBalancerFeignClient实现了OpenFeign的Client接口，负责OpenFeign网络请求的发送和响应的接收，并带有客户端负载均衡机制。loadBalance方法实现如下所示：

```java
/FeignClientFactoryBean.java
protected 〈T〉 T loadBalance(Feign.Builder builder, FeignContext context,
        HardCodedTarget〈T〉 target) {
    //会得到'LoadBalancerFeignClient'
    Client client = getOptional(context, Client.class);
    if (client != null) {
        builder.client(client);
        Targeter targeter = get(context, Targeter.class);
        return targeter.target(this, builder, context, target);
    }
}

```

LoadBalancerFeignClient#execute方法会将普通的Request对象转化为RibbonRequest，并使用FeignLoadBalancer实例来发送RibbonRequest。execute方法会首先将Request的URL转化为对应的服务名称，然后构造出RibbonRequest对象，接着调用lbClient方法来生成FeignLoadBalancer实例，最后调用FeignLoadBalancer实例的executeWithLoadBalancer方法来处理网络请求。LoadBalancerFeignClient#execute方法的具体实现如下所示：

```java
//LoadBalancerFeignClient.java
public Response execute(Request request, Request.Options options) throws IOException {
    try {
        //负载均衡时，host就是需要调用的服务的名称
        URI asUri = URI.create(request.url());
        String clientName = asUri.getHost();
        URI uriWithoutHost = cleanUrl(request.url(), clientName);
        //构造RibbonRequest,delegate一般就是真正发送网络请求的客户端，比如说OkHttpClient和ApacheClient
        FeignLoadBalancer.RibbonRequest ribbonRequest = new FeignLoadBalancer.RibbonRequest(
                this.delegate, request, uriWithoutHost);
        IClientConfig requestConfig = getClientConfig(options, clientName);
        //executeWithLoadBalancer是进行负载均衡的关键
        return lbClient(clientName).executeWithLoadBalancer(ribbonRequest,
                requestConfig).toResponse();
    }
    catch (ClientException e) {
        IOException io = findIOException(e);
        if (io != null) {
            throw io;
        }
        throw new RuntimeException(e);
    }
}
private FeignLoadBalancer lbClient(String clientName) {
    //调用CachingSpringLoadBalancerFactory类的create方法。
    return this.lbClientFactory.create(clientName);
}
```

ILoadBalancer是Ribbon的关键类之一，它是定义负载均衡操作过程的接口。Ribbon通过SpringClientFactory工厂类的getLoadBalancer方法可以获取ILoadBalancer实例。根据Ribbon的组件实例化机制，ILoadBalnacer实例是在RibbonAutoConfiguration中被创建生成的。
SpringClientFactory中的实例都是RibbonClientConfiguration或者自定义Configuration配置类创建的Bean实例。RibbonClientConfiguration还创建了IRule、IPing和ServerList等相关组件的实例。使用者可以通过自定义配置类给出上述几个组件的不同实例。

IRule是定义Ribbon负载均衡策略的接口，你可以通过实现该接口来自定义自己的负载均衡策略，RibbonClientConfiguration配置类则会给出IRule的默认实例。IRule接口的choose方法就是从一堆服务器中根据一定规则选出一个服务器。IRule有很多默认的实现类，这些实现类根据不同的算法和逻辑来进行负载均衡。

只读数据库的负载均衡实现

你需要定义DBServer类来继承Ribbon的Server类，用于存储只读数据库服务器的状态信息，比如说IP地址、数据库连接数、平均请求响应时间等，然后定义一个DBLoadBalancer来继承BaseLoadBalancer类。下述示例代码通过WeightedResponseTimeRule对DBServer列表进行负载均衡选择，然后使用自定义的DBPing来检测数据库是否可用。示例代码如下所示：

```java
public DBLoadBalancer buildFixedDBServerListLoadBalancer(List〈DBServer〉 servers) {
        IRule rule = new WeightedResponseTimeRule();
        IPing ping = new DBPing();
        DBLoadBalancer lb = new DBLoadBalancer(config, rule, ping);
        lb.setServersList(servers);
        return lb;
}


public class DBConnectionLoadBalancer {
    private final ILoadBalancer loadBalancer;
    private final RetryHandler retryHandler = new DefaultLoadBalancerRetryHandler (0, 1, true);
    public DBConnectionLoadBalancer(List〈DBServer〉 serverList) {
        loadBalancer = LoadBalancerBuilder.newBuilder().buildFixedDBServerListLoadBalancer(serverList);
    }
    public String executeSQL(final String sql) throws Exception {
        //使用LoadBalancerCommand来进行负载均衡，具体策略可以在Builder中进行设置
        return LoadBalancerCommand.〈String〉builder()
            .withLoadBalancer(loadBalancer)
            .build()
            .submit(new ServerOperation〈String〉() {
                @Override
                public Observable〈String〉 call(Server server) {
                    URL url;
                    try {
                        return Observable.just(DBManager.execute(server, sql));
                    } catch (Exception e) {
                        return Observable.error(e);
                    }
                }
            }).toBlocking().first();
    }
}

```


