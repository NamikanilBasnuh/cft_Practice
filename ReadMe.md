1)AwsTemplateFormatVersion --> THIS IS A STANDARD KEY --> Google it paste it --> Organize it if you need to do.
2)Under AwsTemplateFormatVersion which is into json format put the "Description": -->This is optional. It gives info what you are doing
3)Now, it is time to use your keyPair(.pem file) We will parameterized it!
 We use "Parameters": {
        "WebServerKey": {   --> this is a logical ID!
            "Description": --> Give any description you want!
            "Type":"AWS::EC2::KeyPair::KeyName"
            }, --> don't forget the comma!!
            **BE CAREFUL WITH OPEN CLOSE CURL BRACES !! 

4)"Resources":{} --> Now, It is time use this. when you create something from scratch , YOU MUST USE this KEY!!
NOTE---> There is a main curl brace!! and inside of the main curl braces we are open new curl braces or we are using new keys

so far this is what it is
{ -->OPENcurlBrace A
    "AWSTemplateFormatVersion": "2010-09-09",
    "Description":""
    "Parameters": {
        "WebServerKey": {
            "Description":"",
            "Type": "AWS::EC2::KeyPair::KeyName"
        }
    },
    "Resources": {  ->open curl brace for resources 
        
        ALL COMPONENTS WILL BE CREATED HERE WITH name and into curl braces 
        as an examp-> "VPC": {},"Public1": {}, etc --> DON'T forget the comma between each 
        
        }  ->close curl brace for resources

}-->CLOSECurlBrace A

5-Create the VPC now -> template link -> https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html   
IMPORTANT: READ REQUIREMENTS CAREFULL AND USE THE KEY VALUE PAIRS WHAT ELSE YOU NEED TO!

"Tags": [                          
    --You are giving the vpc name into "Tags":[{"Key":"NAME,"Value": "YouWriteWhatNameYouWantTO"}]
                    {
                        "Key": "Name",
                        "Value": "YouWriteWhatNameYouWantTO"
                    }

6-Now put a comma after the close curl Braces and start creating you Subnets!!!  
  Template Link ->https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html

  -->"MapPublicIpOnLaunch" : true, --> IT DOES ENABLE OR DISABLE AUTO ASSIGN PUBLIC IPv4 ADDRESS!

NOTE: WE MUST referance our vpc into our subnets to DO CONNECTION! Otherwise Subnets are not going to be connected to the VPC!!!          

7-Copy and Paste second public subnet into the json array. You must need to change availability zones,name,cdirBlock.

******************NOW, Let's go the aws UI and check it out it is woking or not!!!*********************
   a.->CloudFormation
   b.->Create stack 
   c.->choose Create template in Designer radioBox
   d.->click Create template in Designer 
   e.->click Template-->IT IS ON THE BOTTOM LEFT. IT IS NEXT To COMPONENTS!
   d.-> Delete everthing and paste your Json array to here!
   e.-> Click reload icon which is right side on the top, next to ? icon
   f.-> If you are having an error message read it carefully and fix it. Otherwise, you will see a diagram which means that you are done if you haven't made a mistake with cidrblocks etc!
   h.-> Click "Check" icon(It is on the left side on the top next to "Close" icon) to validate your json array
        YOU WILL SEE A pop-up Message says that "Template is Valid"
   I.->click Cloud icon(it is next to "Check" icon) in order to create your stack!!
   J.Now, your stack is inside of "Amazon S3 URL" --> Click next and give a Name

   *************** IT WORKED WELL. IT IS UP AND RUNNING BUT THERE IS A PROBLEM ***********************
   The problem is subnets are connected to only one region! What if the connected aws region is crashed/Down/Not working... our application will stop working too!!! 
   For the Disaster recovery tool / Zero down time --> we will use "Dynamic AZ(Availabity zones)"
   -search for cft az dynamic on google --> Fn::GetAZs --> GezAZs is a ready Cloud Formation Function.Ready to use!
Link->https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-getavailabilityzones.html

--Go down on the link you will see a usage example with subnets into a Json format. copy it and paste it in to your subnets!!

8- I have created a new File ->cft_CreateVpc_DynamicAZ-> And use dynamic az's there. 
     {
      "Fn::Select" : [ 
        "0",            --------> I use "0" or "1" for the ones that I want them to be in the same available zones!
                        --------> I chose my region as nort virginia -> 0 means first index-> means "us-east-1a"
                        --------> Why I am passing this index instead of directly writing "us-east-1a"?
                                  BECAUSE : I WANT TO USE same CFT/Stack for several different regions(Cali,Ohio etc)
                                            AND I DON"T WANT TO KEEP WRITING THEM OVER AND OVER!
                                            THIS IS MORE REUSABLE and GREAT AUTOMATION!
        { 
          "Fn::GetAZs" : "" 
        } 
      ]
    }

 9- Vpc and public subnets are done. Now I am creating 2 private subnets!   
   Private subnets --> "MapPublicIpOnLaunch" : false  MUST BE FALSE!!!!!

10- Create "InternetGateway" and "InternetGatewayAttachment" --> makes VPC and internetGateway attach each other!

11- Route table -> https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-routetable.html

12-Route table configuration->https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-route.html

PublicRouteConfiguration" -->"DependsOn":"InternetGatewayAttachment" it makes ->Wait until internetGateway created!

13-Now, It is time to Associations --> You must associate your subnets to your Route Table!!!!
Link:https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnetroutetableassociation.html
    2 public subnets' associations are done ->"PublicSubnet1RouteTableAssociation"
                                              "PublicSubnet2RouteTableAssociation"


14-Now, According to the diagram, create a NatGateway
link:link:https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-natgateway.html

 "AllocationId" : { "Fn::GetAtt": ["NatElasticIP","AllocationId"]}, -> will explain later

15-Now, Create an elastic IP Because your NatGateway is Public ->You must Allocate an elastic IP!! You were doing the same thing on Amazon UI too!!!!!     
"NATGatewayEIP" : {
   "Type" : "AWS::EC2::EIP",
   "Properties" : {
      "Domain" : "vpc"
   }
}                                      