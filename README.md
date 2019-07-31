# GBDX Task Python interface
 
## Usage

```python
import os
import shutil
from gbdx_task_interface import GbdxTaskInterface

# Derive your task class from the GbdxTaskInterface class
class MyTask(GbdxTaskInterface):
    
    # Define the invoke() method. This will be called below to do the work of the task.
    def invoke(self):

        # Use convenience methods to read input directory/string ports
        input_string_port_value = self.get_input_string_port("input_string_port_name", None)
        input_directory_port = self.get_input_data_port("input_directory_port_name")

        # Write output string ports
        self.set_output_string_port("output_string_port_name", input_string_port_value)

        # Write to an output directory port
        output_data_port = self.get_output_data_port('output dir')
        if not os.path.exists(output_data_port):
            os.mkdir(output_data_port)

        # Copies all the input files to the output.
        # Replace this with actual task code...
        src_files = os.listdir(input_directory_port)
        for file_name in src_files:
            full_file_name = os.path.join(input_directory_port, file_name)
            if os.path.isfile(full_file_name):
                shutil.copy(full_file_name, output_data_port)

        # if the task throws the task will be marked failed and the error message added as the status note
        # raise (Exception("Error message status note"))

        # you can set the status "note" by adding to the reason
        self.reason = 'Task success status note'

        return


if __name__ == "__main__":
    with MyTask() as task:
        task.invoke()

```

## Development and Debugging
When developping or debugging it can be helpful to "run" the task locally.
You can locally create a folder that will mimic the input structure that the task will see when it is run.
You can then set this task interface to use this local folder.

__Create the derived GbdxTaskInterface with the local folder location__
```
MyTask(work_path="/local/folder/location")
```

Then when you locally run the task it will read from and write to this directory.
Note: you will need to remove this work_path from the task when the task is run on the workflow system.

### Create a local workflow task location
To run the above local folder you need to setup the data like the workflow system does
which includes the following

For details see: https://gbdxdocs.digitalglobe.com/docs/task-and-workflow-course


```
/
└── local/folder/location
    ├── gbdx_runtime.json (use to mimic getting tokens from user impersonation)
    ├── input
    |   ├── ports.json (if you use string ports)
    |   └── input_directory_port (one directory per input dir port)
    |       ├── file1.txt
    |       ├── file2.txt
    |       └── file3.txt
    └── output (create this folder but leave empty)
```

#### gbdx_runtime.json
Run time information like the passed token used by user impersonation.  
Contents should look something like this:
```
{
    "user_token":"FAKE_TOKEN"
}
```

#### ports.json
Place your input sting ports in this file inside the input directory
```
{
	"input_string_port_name": "really useful data!"
}
```

#### Create input directory ports
Create folders named after the input directory ports and fill them with your input data files


### After you run your task
You should see your output folder populated and a status.json file created.  If you output any string ports there should be a ports.json file created in the output folder.
For each output directory port there should be a folder created with the generated files.
```
/
└── local/folder/location
    ├── gbdx_runtime.json (use to mimic getting tokens from user impersonation)
    ├── input
    |   ├── ports.json (if you use string ports)
    |   └── input_directory_port (one directory per input dir port)
    |       ├── file1.txt
    |       ├── file2.txt
    |       └── file3.txt
    └── output
    |    ├── ports.json (if you output string ports)
    |    └── output_directory_port (one directory per outputput dir port)
    |       ├── file1.txt
    |       ├── file2.txt
    |       └── file3.txt
    └── status.json    
```

#### status.json
This file is creted on task completion and contains the final status and the "note"
```
{
    "status": "success", 
    "reason": "Task succes status note"
}
```

