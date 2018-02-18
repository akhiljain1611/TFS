# 

       
    private List<WorkItemViewModel> GetWorkItemTree(string query)
            {
                var treeQuery = new Query(_workitemstore, query);
                var links = treeQuery.RunLinkQuery();
    
                var workItemIds = links.Select(l => l.TargetId).ToArray();
    
                query = "SELECT * FROM WorkItems";
                var flatQuery = new Query(_workitemstore, query, workItemIds);
                var workItemCollection = flatQuery.RunQuery();
    
                var workItemList = new List<WorkItemViewModel>();
    
                for (int i = 0; i < workItemCollection.Count; i++)
                {
                    var workItem = workItemCollection[i];
    
                    if (workItem.Type.Name == "Feature")
                    {
                        var model = new WorkItemViewModel()
                        {
                            Id = workItem.Id,
                            AssignedTo = workItem.Fields["Assigned To"].Value.ToString(),
                            Title = workItem.Title,
                            WorkitemType = workItem.Type.Name,
                            Priorty = workItem.Fields["Priority"].Value.ToString(),
                            IterationPath = workItem.IterationPath,
                            State = workItem.State
                        };
    
                        workItemList.Add(model);
                    }
                    else
                    {
                        switch (workItem.Type.Name)
                        {
                            case "Task":
                                var task = new TFSTask()
                                {
                                    ID=workItem.Id,
                                    name = workItem.Title,
                                    //activity = workItem.Fields["MyCompany.Activity"].Value.ToString(),
                                    //start = (DateTime?)workItem.Fields["MyCompany.ActivityStart"].Value,
                                    //due = (DateTime?)workItem.Fields["MyCompany.ActivityFinish"].Value,
                                   status = workItem.State,
                                    IterationPath= workItem.IterationPath,
                                       Assignedto = workItem.Fields["Assigned To"].Value.ToString(),
    
                                    priorty = Convert.ToInt32( workItem.Fields["Priority"].Value),
                                    effort = Convert.ToInt32( workItem.Fields["Estimated Effort"].Value),
                                    Completed=Convert.ToInt32( workItem.Fields["Completed"].Value)
                                   
                                };
    
                                workItemList.Last().Tasks.Add(task);
                                break;
                            case "Bug":
                                var bug = new TFSIssue()
                                {
    
                                    ID = workItem.Id,
                                    Name = workItem.Title,
                                    //start = (DateTime?)workItem.Fields["MyCompany.ActivityStart"].Value,
                                    //due = (DateTime?)workItem.Fields["MyCompany.ActivityFinish"].Value,
                                    State = workItem.State,
                                    IterationPath = workItem.IterationPath,
                                    Assignedto = workItem.Fields["Assigned To"].Value.ToString(),
    
                                    priorty = Convert.ToInt32( workItem.Fields["Priority"].Value),
                                    effort =Convert.ToInt32( workItem.Fields["Estimated Effort"].Value),
                                    Completed = Convert.ToInt32(workItem.Fields["Completed"].Value)
                                };
                                workItemList.Last().Issues.Add(bug);
                                break;
                            case "Product Backlog Item":
                                var backlogitem = new Backlogitem()
                                {
                                    ID = workItem.Id,
                                    Name = workItem.Title,
                                    State = workItem.State,
    
                                    priorty = Convert.ToInt32(   workItem.Fields["Priority"].Value),
                                    //   Size =(int) workItem.Fields["Size"].Value ,
                                      Size  = Convert.ToInt32( workItem.Fields["Effort"].Value),
    
                                    StoryPoints = Convert.ToInt32( workItem.Fields["Story Points"].Value),
                                    DoneStatus=workItem.Fields["Done Status"].Value.ToString(),
                                    StoryOwner=workItem.Fields["Story Owner"].Value.ToString(),
                                    Assignedto=workItem.Fields["Assigned To"].Value.ToString(),
                                    StoryAuthor=workItem.Fields["Story Author"].Value.ToString(),
                                    IterationPath = workItem.IterationPath,
                                };
    
                                workItemList.Last().PBacklog.Add(backlogitem);
                                break;
    
                            default:
                                break;
                        }
                    }
                }
    
                return workItemList;
            }
