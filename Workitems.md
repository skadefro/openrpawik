# Workitems
Inside OpenRPA or OpenFlow you can create a Workitem Queue. A queue can contain an unlimited number of Workitems. Each Workitem represents some data ( and some times files ) that needs to be processed some how. The queue will handle giving out Workitems to clients that asks for a new item to process, conforming to special rules, like how many times can an item be tried before failing completely, in what order they should be process or if an Workitem should be post pone for an amount of time.
You can add Workitems from OpenRPA, in the openflow web interface or from within NodeRED. If you need to add a lot of items, you should use "Buld add" ( called Add workitems in NodeRED ) for better performance.
You can pop items from OpenRPA and NodeRED. ( pop means asking OpenFlow if there is an item ready to be processed, if there is, it will checkout that item to you, and update it's state to "Proccessing". It is now you responsibility to ensure the item changes state to "Retry" or "Successful" using Update Workitem.
If a Workitems fails, you can easily retry that item by clicking the Resubmit 
![image](https://user-images.githubusercontent.com/4155937/163803342-a964ef80-e11f-4048-95e1-2dba82244cce.png) button inside OpenFlow. 

A common practice is to create a "Main processing" workflow that pop items of an queue, and calls a "processing" workflow using Invoke OpenRPA.   
![image](https://user-images.githubusercontent.com/4155937/163803744-06cf703f-be00-4ec5-83e8-21f85db40053.png)  
By encapsulating this activity in a Try Catch activity, we can guarantee we always update the Workitem status to either Successful or Retry.  
![image](https://user-images.githubusercontent.com/4155937/163803802-75da0e10-2adf-4ceb-8ce9-1dc5acec7af5.png)
![image](https://user-images.githubusercontent.com/4155937/163803841-71ffe1d0-5dec-4195-a795-7957d62588ea.png)  

If needed we can easily add a call to a separate Workflow that "prepares" everything ( Ensure needed applications are open and on the right tab etc. )
and add calls to a clean up workflow, that makes sure any error dialogs are closed, kill unresponsive applications, close down chrome etc.  
![image](https://user-images.githubusercontent.com/4155937/163804408-3a558e54-d16e-43ea-a2b5-2f3c071d9c48.png)

If you have a role with RPA enabled with multiple robots, you can assign this role to the project, and the Workitem Queue.  
On the Workitem queue you then set the role as robot target, and the main workflow as entry point workflow, and you can now scale out the workload handling to as may robot as you want ( preferably using [HD Robots](https://www.youtube.com/watch?v=VMQtr0fK3Rw) )

[![WorkItems in OpenRPA part 1](https://img.youtube.com/vi/_y9HU_XPD9c/2.jpg)](https://youtu.be/_y9HU_XPD9c)WorkItems in openrpa part 1  
[![WorkItems in OpenRPA part 2](https://img.youtube.com/vi/B9BnbrhG1yg/2.jpg)](https://youtu.be/B9BnbrhG1yg)WorkItems in OpenRPA part 2  

## Multiple stages
Not all Workitems can or should be processed in one go. Often a unit of work will go though several steps. We can easily achieve this by adding one Workitem Queues for each step, and after a Workitem has compleed successful, we add a new Workitem in the next queue copying the values and files from the original item.  
For instance, say you create a Workitem for each invoice you receive. You process each invoice, like adding it to your local ERP system, you could then create a Workitem with a "nextrun" set to the payment due date of the invoice, to allow a workflow handle all payments at the correct time.  
![image](https://user-images.githubusercontent.com/4155937/163805797-31414b9e-813c-4ee9-b01b-d42ff5454cc7.png)

## OpenRPA video examples
You can find a few videos and example workflows at the following link  
[add-workitems-openrpa](https://github.com/open-rpa/examples-files/tree/master/add-workitems-openrpa)

# OpenRPA Work Item Activities

## Add Workitem
![image](https://user-images.githubusercontent.com/4155937/163792775-fc7c96c0-1208-4d53-9063-b6a73289fba0.png)  
Add new Workitem to the selected Workitem Queue. You can add an unlimited number of parameters using the Payload Property. You can also add as many files as you want, by setting "Files" to an array for strings with the full path and filename to the files you want to attach to the Workitem.
You can schedule this Workitem to not be processed until a specific date/time by setting Next Run. 
If some items are more important than others and you often have many work items in status "new", you can set Priority to prioritize them items. ( the lower the number, the more important it is )

## Bulk Add Workitem
![image](https://user-images.githubusercontent.com/4155937/163792813-841cdfbd-548b-445d-bbd2-4364d92e8cbe.png)  
When adding more than one Workitem it can be more efficient to create all items in one go. Bulk add, creates items based on a data table. Each row in the data table gets created as a new Workitem. To add files for each Workitem set "File Columns" to an array of the column names that will contain the fill path and filename for file(s) to attach to each Workitem. Each column in the data table will get mapped to the payload of each new Workitem

## Delete Workitem
![image](https://user-images.githubusercontent.com/4155937/163792871-a9de5412-1474-410c-953d-08046b624839.png)  
You should never need to use this. This will delete an item from the Workitem Queue and remove any associated files from the database.

## Pop Workitem
![image](https://user-images.githubusercontent.com/4155937/163792955-f01abac0-2bd4-4b6e-91d8-54f4074a9aab.png)  
Takes the next item, ready to be processed. Openflow will handle prioritized items, and filter out items scheduled to run in the future. Once an item has been popped it gets checked out to the current user, and set to state "Processing" and can no longer to popped. So it is important update the item to Successful or Retry once completed. The simple way to do this, is to process the Workitem in a sperate workflow and call "Invoke OpenRPA" inside a try catch activity. 

## Update Workitem
![image](https://user-images.githubusercontent.com/4155937/163793002-e1745656-c6dd-4829-980a-4b70571d8acc.png)  
Updates an Workitem. To the given state, and update the payload based on your changes to the payload property of the Workitem.
Generally you should never set state to failed, but set the state to "Retry" and let openflow decided what to do.
To add or update files associated with this work item set "Files" property to an array of files with the full path and filename.
If Exception is of type Business Rule Exception, the exception type will be set to "business" instead of the default "application" type.
If Exception is of type Business Rule Exception the Workitem will not be re scheduled,

## Throw Business Rule Exception
![image](https://user-images.githubusercontent.com/4155937/163794461-4aefaac2-ce49-4093-b4e7-2f5bcafc44e9.png)  
Throw a new Exception that ends the workflow. By using this exception type we can catch the type in our "main workflow" and set exception type to business instead of the default application exception type. When ever an work item fails, we set state to "retry" and openflow will re scheduled the Work Item to try again ( until maxretries have been hit ) but we will never retry a Business Rule Exception since we already know this item will not succussed if we simply try it again.

# NodeRED nodes
## Configuration node
![image](https://user-images.githubusercontent.com/4155937/164935418-900671bf-5461-4977-b432-3a0a3ff7bc64.png)  
All WorkItem nodes needs a config node, defining what queue they are working with  
Right now you manually have to add the id or queue name, at some point we might add a dropdown list instead  
![image](https://user-images.githubusercontent.com/4155937/164935663-90d71827-6def-401c-aa4f-9472e5c8b058.png)  

## Schedulering processing Workitems
You can setup a cron job, are scheduler to pop items, or you can add an amqp consumer, and set this consumer on an Workitem queue
This will all for a simple workflow like this, that calls a subflow for processing.  
![image](https://user-images.githubusercontent.com/4155937/164965765-f0fbd63f-c284-4d69-bbf5-8905e54747a4.png)
we can the distribute this workflow across multiple NodeRED for easy scaling out our workflow 

## Multiple NodeRED stages
![image](https://user-images.githubusercontent.com/4155937/164966245-e3564b5e-6df9-4fd4-9593-e2195613bb84.png)  
Often you will split up processing into multiple steps. This way we can easily scale out certain parts, it can also make handling retries more simple and resource/time consuming having each logical step separated. We can achieve this simply by creating a Workitem Queue per step and "push" the Workitem forward after each step. Like in the picture above. We can the move the second step to a seperate NodeRED and/or create multiple NodeRED's with this workflow to scale out the Workitem handling

## Add Workitem node
![image](https://user-images.githubusercontent.com/4155937/164935130-81ccb186-8cf0-484f-baa2-092ebb574b7c.png)  
Add a new Workitem to the selected Workitem Queue. By default msg.payload gets added as the payload to the new Workitem.  
You can supply an array for "file objects" to the files property to attach one or more files to the work item.  
Imaging you read a file on disk, or downloaded a file using the http node and have the file content buffer in msg.payload  
![image](https://user-images.githubusercontent.com/4155937/164936449-d9745090-ab04-4b7b-9a3d-899ba25e0195.png)  
We can add that to the workitem using  
```
msg.files = [];
msg.files.push( {
    file: msg.payload,
    filename: "kitten.jpg"
})
```  
nextrun takes a date, if this date is in the future, the workiem will not get processed until this time.  
Priority can be used to prioritize if many items are in the queue. The lower the number the more important the item is.    
msg.topic will set the name of the new Workitem  

## Add Workitems node
![image](https://user-images.githubusercontent.com/4155937/164965395-21f495a3-860c-4908-9d69-1c30e3d932fa.png)  
Takes an array of payloads, and creates a workitem per payload  
Allows setting a Nextrun and Priority for all items as well

## Pop Workitem node
![image](https://user-images.githubusercontent.com/4155937/164938733-6d221629-1792-4b46-baa1-a0cd3e375fb4.png)    
Takes the next item, ready to be processed of the queue. Openflow will handle prioritized items, and filter out items scheduled to run in the future. Once an item has been popped it gets checked out to the current user, and set to state "Processing" and can no longer to popped. So it is important update the item to Successful or Retry once completed. The simple way to do this is to keep the workflow on it's own tab inside NodeRED. Then add a Catch node, set to catch errors from all nodes, and then add an update Workitem node, to set status to Retry.  
![image](https://user-images.githubusercontent.com/4155937/164939433-0f5531e2-4b28-42ac-bf3e-20901fc3b6d4.png)  

## Update Workitem node
![image](https://user-images.githubusercontent.com/4155937/164936942-d26e9b36-e01b-4419-a62d-d4abd98e6644.png)  
Updates the selected Workitem, gotten using Pop Workitem.  
You can add or update files associated with the Workitem by using same syntax as with Add Workitem using an array of "file objects" set to the files property.  
If processing the item, was a success, status should by "Successful", if something went wrong, status should always be set to "Retry" then openflow will determine if, how and when to retry the Workitem based on the Workitem Queue settings.  
If retrying an item, that has already passed the Max retries on the queue, you can force the item to still be re-processed by setting Ignore max retries to true.

## Delete Workitem node
![image](https://user-images.githubusercontent.com/4155937/164965528-cf8b2c88-3296-4667-a1ce-c8f13513a38f.png)  
You should never need to use this, but this node allows you to delete a payload received by the Pop Workitem node