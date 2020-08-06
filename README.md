# Advanced Apex Programming 4th Edition

## Dev, Build and Test

Refer to the Salesforce documentation for Salesforce DX.
This is a Salesforce DX project.
Chapters are located in individual branches in the git repository. Steps within a chapter are represented by individual commits

## Resources

Refer to the license or license.txt file for copyright and license information.

## Description of Files and Directories

Refer to Salesforce DX and Salesforce metadata documetnation for a description of
all files.
For an in-depth explanation of this project, refer to the 4th edition of the book Advanced Apex Programming

## Issues


## =============================================
* triggers can receive up to 200 objects at once
* SOQL queries and DML statements should not be inside of loops
* Apex code should be designed to handle bulk operations

* Why test?
    * Does the code work
    * Code coverage
    * Handling invalid input

One common design pattern is to create utility functions that can be shared by multiple test classes or methods.

## Example on how to use utility functions
in the following example, there are two utility functions.  The first one, `TestBulkPatterns.InitTestObjects`, sets up test scenarios based on some parameters.  The `newopportunities` parameter references a list that is initialized by this function with the new opportunities.  `NumberOfOtherOpportunities` specifies 