# Get tab
![image](https://user-images.githubusercontent.com/4155937/163790922-883fabe5-7686-4de2-869a-7e4f60829ca5.png)  
Using Result property will return the current tab open  
Using Results property will return an array with all open tabs  
By default uses chrome for Result, and all browser for "Results"  
To filer or set browser type, set "Browser" property to "chrome", "ff" or "edge"  

# Continue
![image](https://user-images.githubusercontent.com/4155937/163791831-be9a33b2-c2ad-45e3-bdeb-ef57721f2c4c.png)  
When used inside a breakable loop ( get element, for each, do while, while etc. ) you can skip the remaining activities and make the loop move to next item.

# Break
![image](https://user-images.githubusercontent.com/4155937/163792032-beb3c41e-e38b-4307-b31c-465bee7e628a.png)  
When used inside a breakable loop ( get element, for each, do while, while etc. ) you can skip the remaining activities and stop processing more items and moves to the next activity after the breakable loop.

# Do While
![image](https://user-images.githubusercontent.com/4155937/163792119-8bf0a053-289a-42cd-948f-77aed30c00b7.png)  
Creates a loop that executes other activities dropped in it at least once and repeatedly until the looping condition no longer evaluates to True.

# While
![image](https://user-images.githubusercontent.com/4155937/163792333-4a34cb54-9b3f-45d4-bc3c-e17fa8e02b28.png)  
This Activity creates a loop that executes other Activities dropped in it while the condition is True. Once condition no longer evaluates to True, the loop ceases and moves to the next Activity. 

# Add Workitem
![image](https://user-images.githubusercontent.com/4155937/163792775-fc7c96c0-1208-4d53-9063-b6a73289fba0.png)  
Add new Workitem to the selected Workitem Queue. You can add an unlimited number of parameters using the Payload Property. You can also add as many files as you want, by setting "Files" to an array for strings with the full path and filename to the files you want to attach to the Workitem.
You can schedule this Workitem to not be processed until a specific date/time by setting Next Run. 
If some items are more important than others and you often have many work items in status "new", you can set Priority to prioritize them items. ( the lower the number, the more important it is )

# Bulk Add Workitem
![image](https://user-images.githubusercontent.com/4155937/163792813-841cdfbd-548b-445d-bbd2-4364d92e8cbe.png)  
When adding more than one Workitem it can be more efficient to create all items in one go. Bulk add, creates items based on a data table. Each row in the data table gets created as a new Workitem. To add files for each Workitem set "File Columns" to an array of the column names that will contain the fill path and filename for file(s) to attach to each Workitem. Each column in the data table will get mapped to the payload of each new Workitem

# Delete Workitem
![image](https://user-images.githubusercontent.com/4155937/163792871-a9de5412-1474-410c-953d-08046b624839.png)  
You should never need to use this. This will delete an item from the Workitem Queue and remove any associated files from the database.

# Pop Workitem
![image](https://user-images.githubusercontent.com/4155937/163792955-f01abac0-2bd4-4b6e-91d8-54f4074a9aab.png)  
Takes the next item, ready to be processed. Openflow will handle prioritized items, and filter out items scheduled to run in the future. Once an item has been popped it gets checked out to the current user, and set in state "Processing" and can no longer to popped. So it is important update the item to Successful or Retry once completed. The simple way to do this, is to process the Workitem in a sperate workflow and call "Invoke OpenRPA" inside a try catch activity. 

# Update Workitem
![image](https://user-images.githubusercontent.com/4155937/163793002-e1745656-c6dd-4829-980a-4b70571d8acc.png)  
Updates an Workitem. To the given state, and update the payload based on your changes to the payload property of the Workitem.
Generally you should never set state to failed, but set the state to "Retry" and let openflow decided what to do.
To add or update files associated with this work item set "Files" property to an array of files with the full path and filename.
If Exception is of type Business Rule Exception, the exception type will be set to "business" instead of the default "application" type.
If Exception is of type Business Rule Exception the Workitem will not be re scheduled,

# Throw Business Rule Exception
![image](https://user-images.githubusercontent.com/4155937/163794461-4aefaac2-ce49-4093-b4e7-2f5bcafc44e9.png)  
Throw a new Exception that ends the workflow. By using this exception type we can catch the type in our "main workflow" and set exception type to business instead of the default application exception type. When ever an work item fails, we set state to "retry" and openflow will re scheduled the Work Item to try again ( until maxretries have been hit ) but we will never retry a Business Rule Exception since we already know this item will not succussed if we simply try it again.
