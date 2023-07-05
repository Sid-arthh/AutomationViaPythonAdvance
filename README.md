# Scaling cloudformation resources dynamically

## About

Welcome to the repository! This project features a Python wrapper script designed to simplify the process of scaling CFM (Cloud Resource Management) resources. By running this script, you can effortlessly manage and deploy resources based on your requirements.
Functionality

### The script provides the following functionalities:
Resource Scaling: It automatically checks if the resource enable flag is set to True. If enabled, the script dynamically creates and adds the specified resource during deployment.

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
        
        python3 main.py parameter.yml

