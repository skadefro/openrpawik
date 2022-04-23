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

## Multiple stages
Not all Workitems can or should be processed in one go. Often a unit of work will go though several steps. We can easily achieve this by adding one Workitem Queues for each step, and after a Workitem has compleed successful, we add a new Workitem in the next queue copying the values and files from the original item.  
For instance, say you create a Workitem for each invoice you receive. You process each invoice, like adding it to your local ERP system, you could then create a Workitem with a "nextrun" set to the payment due date of the invoice, to allow a workflow handle all payments at the correct time.  
![image](https://user-images.githubusercontent.com/4155937/163805797-31414b9e-813c-4ee9-b01b-d42ff5454cc7.png)

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
Takes the next item, ready to be processed. Openflow will handle prioritized items, and filter out items scheduled to run in the future. Once an item has been popped it gets checked out to the current user, and set in state "Processing" and can no longer to popped. So it is important update the item to Successful or Retry once completed. The simple way to do this, is to process the Workitem in a sperate workflow and call "Invoke OpenRPA" inside a try catch activity. 

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

## API Add Workitem
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
