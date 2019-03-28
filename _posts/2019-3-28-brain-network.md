---
title: "脑电脑网络分析代码使用流程介绍"
layout: post
category: EEG
tags: [matlab脚本,EEG,脑网络分析]
excerpt: "本文主要就是对之前我在徐老师组学习期间使用过的代码的一个整理，大家以后如果需要做脑网络分析的，可以根据文中描述的步骤使用这些代码"
---
2018年5月-7月，袁老师安排我去电子科大跟随徐老师学习脑电数据挖掘，徐老师组主要用的是EEG脑网络分析方法，因此在科大的两个多月的时间里主要就是学习了如何用matlab脚本来进行脑电的脑网络。本文主要就是对之前我在徐老师组学习期间使用过的代码的一个整理，大家以后如果需要做脑网络分析的，可以根据文中描述的步骤使用这些代码（代码都放在实验室的私有云上面了，路径：` /Documents/ERP and EEG/ACR脑电数据处理代码集/4.EEG脑网络分析代码`）。

# 1.预处理
此处的预处理不是针对原始数据的预处理，而是对已经预处理完的数据而言的。代买如下：

```matlab

 clear all
 close all
 clc
% path = 'C:\Users\Administrator\Desktop\ERP_yukang';
% filename = dir(path);%加载路径（文件）
 File = 'D:\61_all\'; %这里改为数据导入路径
 savepath = 'D:\61_conditions\' ;%这里改为数据导出路径
% freqband = [0.02 40];
% freqband = [0.02 5];
 max_order = 99;
 for sub = 1:61;%这里改为自己数据中被试的数量
    %     cnt = loadcnt1([path, '\', filename(sub).name]);%加载cnt数据
    %     a = pop_loadset('C:\Users\Administrator\Desktop\ERP_yukang\sub1_erp.set');
%     eval(['pop_loadset',' ',File,'sub',num2str(sub),'_erp.set']); %加载rest数据
%     eval(['pop_loadset',' ',File,'sub_0',num2str(sub),'.mat']); %加载rest数据
      if sub<= 9;
           eval(['pop_loadset',' ',File,'sub_0',num2str(sub),'.mat']); %加载rest数据
    else
      eval(['pop_loadset',' ',File,'sub_',num2str(sub),'.mat']);
    end     
    struct = ans;
    Data1 = ans.data;
    %% 去导联,得到准确的数据
%     [bool,inx] = ismember ({'AF3','Fz','F1','F3','FC1','F7','FCz','FC3','Cz','C1','C3','CP3','Pz','P3','P5','P7','PO3','PO7','Oz','O1','AF4','F2','F4','F8','FC2','FC4','C2','C4','CPz','CP4','P4','P6','P8','POz','PO4','PO8','O2'},{struct.chanlocs.labels}.');%inx是位置，bool是布尔值,
    [bool,inx] = ismember ({'AF3','Fz','F1','F3','FCz','Cz','C1','C3','CP1','CP3','Pz','P3','Oz','O1','AF4','F2','F4','C2','C4','CPz','CP2','CP4','P4','POz','O2'},{struct.chanlocs.labels}.');%inx是位置，bool是布尔值,
   Data = Data1([inx],:,:);
    stimtype = [];%
    offset = [];%
    num1 = 0; num2 = 0; num3 = 0;num4 = 0;
    C_V_data = []; F_N_data = []; F_data = [];S_data = [];
    %获取标签值 free_neutral    change_suppression    free    suppresion   change_view
    for  ii = 1:length(struct.event);
        if strncmp(struct.event(ii).type,'change_view',11)
                num1 = num1 + 1;
               C_V_data(:,:,num1) =  Data(:,:,ii) ;%change_view_data
        end
        if strncmp(struct.event(ii).type,'free_neutral',12)
                num2 = num2 + 1;
               F_N_data(:,:,num2) =  Data(:,:,ii); %free_neutral_data
        end
        if strncmp(struct.event(ii).type,'free',11)
            num3 = num3+1;
            F_data(:,:,num3) =  Data(:,:,ii);
        end

        if strncmp(struct.event(ii).type,'suppresion',11)
            num4 = num4+1;
            S_data(:,:,num4) =  Data(:,:,ii);%suppresion_data
        end
    end

    if sub<= 9;
           save([savepath,'C_V_data','\','CVdata_0',num2str(sub)],'C_V_data');
           save([savepath,'F_N_data','\','FNdata_0',num2str(sub)],'F_N_data');
           save([savepath,'F_data','\','Fdata_0',num2str(sub)],'F_data');
           save([savepath,'S_data','\','Sdata_0',num2str(sub)],'S_data');
       else
           save([savepath,'C_V_data','\','CVdata_',num2str(sub)],'C_V_data');
           save([savepath,'F_N_data','\','FNdata_',num2str(sub)],'F_N_data');
           save([savepath,'F_data','\','Fdata_',num2str(sub)],'F_data');
           save([savepath,'S_data','\','Sdata_',num2str(sub)],'S_data');
    end

end

```
上述代码有些地方需要自己修改，除数据储存路径需要修改之外，还有几个地方需要强调：（1）在导入文件时，对被试的编号有一个判断语句，这是因为我的数据命名时，对1-9号被试采用的是01、02....09这种命名方式，因此在循环导入被试数据时，1-9的被试和10以后的被试的导入方式被分开了，如果你的数据命名方式不一样，那么此处需要改动。（2）代码中有一个 `ismenber` 的函数，这个函数是用来判断当前进入循环的被试是否包含大括号中的所有电极点，返回值中有一个bool值，如果该被试缺少括号中的电极，那么bool值就为0，否则为1，当bool值中出现了0，程序就会报错并中断，这个时候我就可以根据bool值反馈的信息对大括号中的电极点进行删改，直到所有的被试都能跑通，有时候需要对电极点和被试的数量做折中处理，当然如果一些非常重要的电极点缺失的话那就需要补电极了，关于补电极点会在后面的内容中提到。（3）
