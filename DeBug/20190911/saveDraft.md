```csharp
  //设置单据状态
                if (!rp.IsSubmit.Value && enableDraft.ToLower().Equals("true"))
                {
                    entity.Status = (int?)BillStatuses.Draft;
                }
                else
                {
                    #region 审批流

                    //判断单据是否有审批流
                    //设置表是否设置启用流程
                    var servicesInfo = ConfigInfo.Get<ServiceLocationsConfigInfo>();
                    var service = servicesInfo.FindBy("Service.System");
                    var billApproverFlow = PostAsync<BillApproverFlow>(
                        service.RootURL + "/BillApproval/CorpApprovalFlow/GetEffectedFlowObj",
                        new
                        {
                            Data = new
                            {
                                userInfo.CorpID,
                                BillType = AssetBillTypes.PurchasingApplication,
                                ProductID = C8.Common.ValueObject.Products.Asset
                            }
                        },
                        pRequest.Token, out string errorMsg);
                    //流程表匹配启用流程
                    if (billApproverFlow != null && billApproverFlow.FlowId.HasValue)
                    {
                        #region 启用流程

                        entity.Status = (int?)BillStatues.Submitted;
                        entity.OutFlowID = billApproverFlow.OutFlowID;
                        entity.FlowID = billApproverFlow.FlowId;

                        #endregion 启用流程
                    }
                    else
                    {
                        #region 未启用流程

                        //如果没有启用流程
                        if (isNeedSign.Equals("True")) //是否开启签字确认
                        {
                            entity.Status = (int?)BillStatues.WaitingConfirm;
                        }
                        else
                        {
                            entity.Status = (int?)BillStatues.Effected;
                            entity.SignDatetime = Clock.Now;
                        }

                        #endregion 未启用流程
                    }

                    #endregion 审批流
                }
```

