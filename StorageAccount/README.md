


**Task : Download the blob from Storage Account - Container using Managed Identity**


To download the blob from SA-Container, please follow below steps. 

**Step 1 : Get the Application id from System Managed identity or Use Managed identity**


![](https://github.com/praveenambati1233/Azure/blob/main/images/sa.PNG)

**Step 2:  Login into subscription using System Managed identity/ User Manged identity**

`$az login --identity -u <Managed Identity Application id >`

**Step 3 : Download the blob file from the Storage Account Container **

`$az storage blob download --auth-mode login --account-name <SA_NAME> --container-name <CONATINER_NAME> --file /YOUR/PAHT.txt --name BlOB_NAME.txt`


