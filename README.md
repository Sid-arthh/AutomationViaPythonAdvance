# Scaling cloudformation resources dynamically

## About

Welcome to the repository! This project features a Python wrapper script designed to simplify the process of scaling CFM (Cloud Resource Management) resources. By running this script, you can effortlessly manage and deploy resources based on your requirements.

### Functionality

### The script provides the following functionalities:
Resource Scaling: It automatically checks if the resource enable flag is set to True. If enabled, the script dynamically creates and adds the specified resource during deployment.

### NOTE 
#### Client file that is ( parameter.yml ) format 

    template: 'ec2'
    description: 'Deploy ec2'
    ec2:
      Type: AWS::EC2::Instance
      ImageId: ami-053b0d53c279acc90
      InstanceType: t2.micro
      TagValue: TestInstance
    securityGroup:
      Type: AWS::EC2::SecurityGroup
      GroupDescription: Allow TCP traffic
      Port: 0
    
    ALB:
      Type: 'AWS::ElasticLoadBalancingV2::ListenerRule'
      TargetGroupType: 'forward'
    
    s3:
      Type: 'AWS::S3::Bucket'
      deletionPolicy: Retain
      bucketName: my-test-bucket 
    
    
    ALBEnabled: 'true'
    s3Enabled: 'true'

1. The yml file must contain the a key "template" with the name of base resource that you want to deploy.

        template : 'Name of the resource'

2. The Resources Key Name in Yml file should be used as Prefix for Enabled key
   
               For ALB, Enable key should be 
               ALBEnabled
               For s3, Enable key should be 
               s3Enabled
   
### Getting Started
To get started with the script, follow these steps:

Clone this repository to your local machine.
Install the necessary dependencies [ python3+ ]
  
Configure the script by modifying the some variables to match your environment.
Run the script using the command python main.py.
Sit back and let the script handle the scaling of CFM resources!

### How to configure and use it


To configure and use the script effectively, follow the steps outlined below:
#### Directory Structure

The project directory consists of two main directories:
1. Resources
The Resources directory holds all the necessary resource files, such as ec2.py, alb.py, and others. Additionally, it contains a parameters.yml file where clients can pass data for the resources.

2. Template
The Template directory contains the main.py file, which serves as the core code responsible for executing all the functionalities. It is important to note that the main.py file can have a different name depending on the resource it manages (e.g., ec2.py, s3.py). Running the main.py file generates the output file required for deploying the CloudFormation stack.
Configuration Steps

#### Follow these steps to configure and use the script effectively:
Clone the repository to your local machine.
Navigate to the project's main directory.

  Resource Configuration:
  
        Inside the Resources directory, locate the specific resource file you want to configure (e.g., ec2.py, alb.py).
        Modify the resource file according to your requirements and desired settings.
        Ensure that the parameters.yml file contains the necessary data to customize the resource.

  Main Code Configuration:
  
        Access the Template directory and open the main.py (or equivalent) file.
        Customize the code within the main.py file to meet your specific needs and desired functionality.

  Run the main.py (or equivalent) file using the appropriate Python command.

After executing the script, an output file will be generated. This output file contains the necessary information for deploying the CloudFormation stack. Ensure you have the required permissions and access to perform the deployment.

#### Command to Run the Script

        locate the main.py in resource directory and run the command
        
        python3 main.py path/to/parameter.yml


### Code Deep Dive
    

In this section, we will provide a detailed explanation of the updated code for better understanding. Let's break down the code into smaller parts and discuss their functionalities.

#### Dependencies and imports

The below lines import the necessary modules for the script, including sys, yaml, os, and importlib
  
    import sys
    import yaml
    import os
    import importlib

#### Variables and Console input 

Here, we define some initial variables. directory represents the path to the "resources" directory. combined_data and combined_data_dict will be used to store the combined data from various resources. resource_names will store the names of the resources. outstream_file indicates the path to the output file. The script checks if a YAML file path is provided as a command-line argument and assigns it to Client_yml_file.

    directory = "./resources"
    combined_data = {}
    combined_data_dict = {}
    resource_names = []
    outstream_file = "resources/outstream.yml"
    parameter_template = read_file(Client_yml_file)
    
    if len(sys.argv) < 2:
        print("Please provide a YAML file path as a command line argument.")
        sys.exit(1)
    Client_yml_file = sys.argv[1]

## Too many Functions 
1.The read_file function reads and returns the content of a YAML file.

    def read_file(file_path):
      with open(file_path, 'r') as file:
            data_content = file.read()
            data = yaml.safe_load(data_content)
      return data

2.The process_template function is responsible for processing the client YAML file. It retrieves the main template file path from the parameter_template data. The function then changes the current working directory, appends the current directory to the system path, and calls the AddResource function with the parameter_template and main_template_file as arguments.

    def process_template(Client_yml_file):
      template_dir = os.path.abspath(Client_yml_file)
      template_value = parameter_template.get('template', '')
      main_template_file = template_value 
      os.chdir('..') 
      sys.path.append(os.getcwd())
      AddResource(parameter_template, main_template_file)

3.The AddResource function processes the data from the client YAML file and identifies the resources that are enabled based on the resource_enabled flag. It adds the main template to the resource_names list and appends other enabled resources as well. The function then prepares the module name and calls the import_Module_data function to process each resource.

    def AddResource(data, main_template):
      resource_names.append(main_template)
      for key, value in data.items():
          resource_enabled = data.get(f'{key}Enabled', '') 
          if isinstance(value, dict) and resource_enabled == 'true': 
              resource_names.append(key.lower())
              print("resource is", resource_names)
  
      for name in resource_names:
          module_name = f"resources.{name}"
          module_name2 = name + '.py' 
          import_Module_data(name, module_name, module_name2)


4.The import_Module_data function is responsible for importing the resource modules and calling their respective create_{name} functions. It iterates over the files in the directory and checks if the file name matches the desired module name. If found, it imports the module, retrieves the create_{name} function, and calls it to obtain the temporary data. The temporary data is then assigned to combined_data, and the resource_Key_check function is called to process the combined data.

    def import_Module_data(name, module_name, module_name2): 
      for file in os.listdir(directory):
          print("file is", file)
          current_dir = os.path.dirname(file)
          if file.lower() == module_name2.lower():
              file_path = os.path.join(current_dir, file)
              file_path = f"resources.{file}"
              resource_name = os.path.splitext(file_path)[0]
  
              try:
                  module = importlib.import_module(resource_name, package="pythons.template")
                  print("module is", module)
                  create_function = getattr(module, f"create_{name.casefold()}")
                  temp_data = create_function()
                  print(temp_data)
                  combined_data = temp_data
                  resource_Key_check(combined_data_dict, combined_data)
  
              except Exception as e:
                  print(f"Error occurred while calling create_{name}() function in {module_name}: {e}")


5.The resource_Key_check function checks if the combined_data_dict already contains a "Resources" or "resources" key. If found, it updates the corresponding key's value by merging the combined_data. Otherwise, it directly updates the combined_data_dict. If there are less than two resources, the combine_and_store function is called to process the combined data.

     def resource_Key_check(combined_data_dict, combined_data):
          if 'Resources' in combined_data_dict or 'resources' in combined_data_dict:
               resources_key = 'Resources' if 'Resources' in combined_data_dict else 'resources'
               combined_data_dict[resources_key].update(combined_data)
              combine_and_store(combined_data_dict)
          else:
              combined_data_dict.update(combined_data)
              if len(resource_names) < 2:
                   combine_and_store(combined_data_dict)

6.The combine_and_store function is responsible for combining the combined_data with the existing data in the outstream_file. If the file already exists and contains the same data as combined_data, the write operation is skipped. Otherwise, the combined data is dumped into the outstream_file using yaml.safe_dump and the call_replace_with_outstream_file function is called.

    def combine_and_store(combined_data):
      if os.path.exists(outstream_file):
          with open(outstream_file, 'r') as file:
              existing_data = yaml.safe_load(file)
  
          if existing_data == combined_data:
              print("Output file already contains the same data. Skipping write operation.")
              call_replace_with_outstream_file()
      else:
          with open(outstream_file, 'a') as outfile:
              yaml.safe_dump(combined_data, outfile, sort_keys=False, default_flow_style=False)
              outfile.write('\n\n')
              call_replace_with_outstream_file()

7.The replace_values function recursively traverses the data structure and replaces string values that contain a period ('.') with corresponding values from the replacements dictionary. It splits the string value into replace_key and replace_value and checks if they exist in the replacements dictionary. If found, it replaces the original value with the corresponding value from the dictionary.

    def replace_values(data, replacements):
      if isinstance(data, dict):
          for key, value in data.items():
              if isinstance(value, str) and '.' in value:
                  parts = value.split('.')
                  replace_key = parts[0]
                  replace_value = parts[1]
                  if replace_key in replacements:
                      value1 = replacements[replace_key]
                      if replace_value in value1:
                          data[key] = data[key].replace(value, str(value1[replace_value]))
                  else:
                      return None
              else:
                  replace_values(value, replacements)
  
      elif isinstance(data, list):
          for item in data:
              replace_values(item, replacements)

8.The call_replace_with_outstream_file function reads the content of the outstream_file using read_file, replaces values using replace_values with the parameter_template, and dumps the formatted content into a new file called 'out.yml'. Finally, it removes the outstream_file.

     def call_replace_with_outstream_file():
      outfile = read_file(outstream_file)
      replace_values(outfile, parameter_template)
      formatted_content = yaml.dump(outfile, sort_keys=False)
      with open('out.yml', 'w') as f:
          f.write(formatted_content)
          os.remove(outstream_file)

9.The script starts the execution by calling the process_template function with the Client_yml_file as an argument.

      process_template(Client_yml_file)

### Overall Functionality
The provided code aims to process a client YAML file containing information about resources and templates. It dynamically imports resource modules based on the enabled resources specified in the YAML file. The code combines the data from various resources and stores it in an output YAML file. It also replaces values in the output YAML file using a parameter_template.

