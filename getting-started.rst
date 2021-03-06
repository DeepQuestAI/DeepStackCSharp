.. DeepStack documentation master file, created by
   sphinx-quickstart on Wed Dec 12 17:30:35 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Getting Started with DeepStack
===============================

DeepStack is distributed as a docker image. In this tutorial, we shall 
go through the complete process of using DeepStack
to build a Face Recognition system.

Setting Up DeepStack
---------------------

Follow instructions on read :ref:`home` to install the CPU Version of DeepStack
If you have a system with Nvidia GPU, follow instruction on read :ref:`gpuinstall` to install the GPU Version of DeepStack

**Installing DeepStack**

DeepStack is an AI Server that provides AI features as APIs consumable via basic web requests.
It works entirely offline and can be installed anywhere docker runs both on premise and in the cloud.

DeepStack is developed and maintained by `DeepQuest AI <https://deepquestai.com />`_

Run the command below to install DeepStack ::
    
    docker pull deepquestai/deepstack

To install the GPU Accelerated Version, follow :ref:`gpuinstall`

**Starting DeepStack**

Below we shall run DeepStack with only the FACE features enabled ::

    sudo docker run -e VISION-FACE=True -v localstorage:/datastore -p 80:5000 deepquestai/deepstack

*Basic Parameters*

    **-e VISION-FACE=True** This enables the face recognition APIs, all apis are disabled by default.

    **-v localstorage:/datastore** This specifies the local volume where deepstack will store all data.

    **-p 80:5000** This makes deepstack accessible via port 80 of the machine.

**NOTE FOR THE GPU VERSION**

    If you installed the GPU Version, remmember to add the args args **--rm --runtime=nvidia** 
    The equivalent run command for the gpu version is ::

        sudo docker run --rm --runtime=nvidia -e VISION-FACE=True -v localstorage:/datastore \
        -p 80:5000 deepquestai/deepstack:gpu
    


**Face Recognition**
    
    Think of a software that can identity known people by their names. Face Recognition does exactly that. Register a picture of a number of people
    and the system will be able to recognize them again anytime.
    Face Recognition is a two step process: The first is to register a known face and second is to recognize unknown faces.

**REGISTERING A FACE**
    
    Here we are building an application that can tell the names of a number of popular celebrities.
    First we collect pictures of a number of celebrities and we register them with deepstack


    .. figure:: cruise.jpg
        :align:  left

    .. figure:: adele.jpg
        :align:  left

    .. figure:: elba.jpg
        :align:  left

    .. figure:: perri.jpg
        :align:  left

Below we will register the faces with their names ::
    
    using System;
    using System.IO;
    using System.Net.Http;
    using System.Threading.Tasks;

    namespace app
    {

    class App {

        static HttpClient client = new HttpClient();

        public static async Task registerFace(string userid, string image_path){

            var request = new MultipartFormDataContent();
            var image_data = File.OpenRead(image_path);
            request.Add(new StreamContent(image_data),"image",Path.GetFileName(image_path));
            request.Add(new StringContent(userid),"userid");
            var output = await client.PostAsync("http://localhost:80/v1/vision/face/register",request);
            var jsonString = await output.Content.ReadAsStringAsync();
            
            Console.WriteLine(jsonString);

        }

        static void Main(string[] args){

            registerFace("Tom Cruise","cruise.jpg").Wait();
            registerFace("Adele","adele.jpg").Wait();
            registerFace("Idris Elba","elba.jpg").Wait();
            registerFace("Christina Perri","perri.jpg").Wait();
            
        }

    }
    
    }

**RECOGNITION**

    Now we shall attempt to recognize any of these celebrities using DeepStack.
    Below we will send in a whole new picture of Adele and DeepStack will attempt to
    predict the name.

    Here we are using the `JSON.NET <https://www.nuget.org/packages/Newtonsoft.Json />`_ package to decode the JSON result.



.. figure:: test-image.jpg
    :align:  left
    

Prediction code ::

    using System;
    using System.IO;
    using System.Net.Http;
    using System.Threading.Tasks;
    using Newtonsoft.Json;


    namespace appone
    {

    class Response {
        
        public bool success {get;set;}
        public Face[] predictions {get;set;}

    }

    class Face {

        public string userid {get;set;}
        public float confidence {get;set;}
        public int y_min {get;set;}
        public int x_min {get;set;}
        public int y_max {get;set;}
        public int x_max {get;set;}
    
    }

    class App {

        static HttpClient client = new HttpClient();

        public static async Task recognizeFace(string image_path){

            var request = new MultipartFormDataContent();
            var image_data = File.OpenRead(image_path);
            request.Add(new StreamContent(image_data),"image",Path.GetFileName(image_path));
            var output = await client.PostAsync("http://localhost:80/v1/vision/face/recognize",request);
            var jsonString = await output.Content.ReadAsStringAsync();
            Response response = JsonConvert.DeserializeObject<Response>(jsonString);
            
            foreach (var user in response.predictions){

                Console.WriteLine(user.userid);

            }

        }

        static void Main(string[] args){

            recognizeFace("test-image.jpg").Wait();

        }

    }
    
    }

Result ::

    Adele
 
.. toctree::
   :maxdepth: 2
   :caption: Contents:

We have just created a face recognition system. You can try with different people and test on different pictures of them.

The next tutorial is dedicated to the full power of the face recognition api as well as best practices to make the best out of it.

**Performance**

DeepStack offers three modes allowing you to tradeoff speed for peformance. 
During startup, you can specify performance mode to be , "High","Medium" and "Low"

The default mode is "Medium"

You can speciy a different mode as seen below ::

    sudo docker run -e MODE=High -e VISION-FACE=True -v localstorage:/datastore \
    -p 80:5000 deepquestai/deepstack

Note the -**e MODE=High** above 

