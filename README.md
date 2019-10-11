# activiti-helloworld
# 新手入门Activit6.0----HelloWorld

#####  使用Eclipse进行流程图绘制
 eclipse上安装activiti的插件，进行绘制流程图。
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011142150185.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rhb2ppbjEy,size_16,color_FFFFFF,t_70)
每个节点设置form信息
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191011142405878.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Rhb2ppbjEy,size_16,color_FFFFFF,t_70)
网关审批结果根据条件进行跳转设置

###### 1、创建流程引擎
创建基于内存数据库的流程引擎
```
  ProcessEngineConfiguration cfg = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
        ProcessEngine processEngine = cfg.buildProcessEngine();
        String name = processEngine.getName();
        String version = ProcessEngine.VERSION;
```
###### 2、部署流程定义文件
将之前在eclipse上绘制的流程图复制在resource文件下，命名为**XX.bpmn20.xml**

```
 RepositoryService repositoryService = processEngine.getRepositoryService();
        DeploymentBuilder deploymentBuilder = repositoryService.createDeployment();
        deploymentBuilder.addClasspathResource("second_approve.bpmn20.xml");
        Deployment deployment = deploymentBuilder.deploy();
        String deploymentId = deployment.getId();
        // 获取流程定义对象
        ProcessDefinition processDefinition = repositoryService.createProcessDefinitionQuery()
                .deploymentId(deploymentId)
                .singleResult();
        logger.info("流程定义文件名称：{},流程id：{}", processDefinition.getName(), processDefinition.getId());
```
##### 3、启动运行流程

```
    RuntimeService runtimeService = processEngine.getRuntimeService();
        ProcessInstance processInstance = runtimeService.startProcessInstanceById(processDefinition.getId());
        logger.info("启动流程：{}", processInstance.getProcessDefinitionKey());
```
##### 4、处理流程任务

```
 Scanner scanner = new Scanner(System.in);
        while (processInstance != null && !processInstance.isEnded()) {
            TaskService taskService = processEngine.getTaskService();
            List<Task> list = taskService.createTaskQuery().list();
            logger.info("待处理任务数量：{}", list.size());
            for (Task task : list) {
                logger.info("待处理任务：{}",task.getName());
                FormService formService = processEngine.getFormService();
                TaskFormData taskFormData = formService.getTaskFormData(task.getId());
                List<FormProperty> formProperties = taskFormData.getFormProperties();
                HashMap<String, Object> variables = Maps.newHashMap();
                for (FormProperty formProperty : formProperties) {
                    String line = null;
                    if (StringFormType.class.isInstance(formProperty.getType())) {
                        logger.info("请输入{}？", formProperty.getName());
                        line = scanner.nextLine();
                        variables.put(formProperty.getId(), line);
                    } else if (DateFormType.class.isInstance(formProperty.getType())) {
                        logger.info("请输入：{} 格式(yyyy-MM-dd)", formProperty.getName());
                        line = scanner.nextLine();
                        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
                        Date date = simpleDateFormat.parse(line);
                        variables.put(formProperty.getId(), date);
                    } else {
                        logger.info("类型暂不支持", formProperty.getType());
                    }
                    logger.info("你输入的内容是：{}", line);
                }
                // 提交工作
                taskService.complete(task.getId(), variables);
                // 执行完一次，重新获取流程实例进行状态判断
                 processInstance = processEngine.getRuntimeService()
                        .createProcessInstanceQuery()
                        .processInstanceId(processInstance.getId())
                        .singleResult();
            }
        }
```

