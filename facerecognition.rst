.. DeepStack documentation master file, created by
   sphinx-quickstart on Wed Dec 12 17:30:35 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

.. _facerecognition:

Face Recognition
=================

In the Getting Started, we had an overview of the face recognition API. In this section, we shall explore all the functionalities 
of the API.

Face Registeration
------------------

The face registeration endpoint allows you to register pictures of person and associate it with a userid.

You can specify multiple pictures per person during registeration.

Example ::

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
            request.Add(new StreamContent(image_data),"image1",Path.GetFileName(image_path));
            request.Add(new StringContent(userid),"userid");
            var output = await client.PostAsync("http://localhost:80/v1/vision/face/register",request);
            var jsonString = await output.Content.ReadAsStringAsync();
            
            Console.WriteLine(jsonString);

        }

        static void Main(string[] args){

            registerFace("User Name ","userimage-path").Wait();
            
        }

    }
    
    }

Result ::

    {'message': 'face added', 'success': True}

The response above indicates the call was successful. You should always check for the "success" status.
If their is an error in your request, you will receive a response like ::

    {'error': 'user id not specified', 'success': False}

This indicates that you ommited the userid in your request.
If you ommited the image, the response will be ::

    {'error': 'No valid image file found', 'success': False}



Face Recognition
-----------------
    
The face registeration endpoint detects all faces in an image and returns the USERID for each face. Note that the USERID was specified
during the registeration phase. If a new face is encountered, the USERID will be unknown. 

We shall test this on the image below.

.. figure:: test-image2.jpg
    :align: center
    


    
::
    
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

            Console.WriteLine(jsonString);

        }

        static void Main(string[] args){

            recognizeFace("test-image2.jpg").Wait();

        }

    }
    
    }


Result ::

    Idris Elba
    unknown
    Full Response:  {'success': True, 'predictions': [{'x_min': 215, 'confidence': 0.76965684, 'x_max': 264, 'y_max': 91, 'y_min': 20, 'userid': 'Idris Elba'}, {'x_min': 115, 'confidence': 0, 'x_max': 162, 'y_max': 97, 'y_min': 31, 'userid': 'unknown'}]}

As you can see above, the first user is unknown since we did not previously register her, however, Idris Elba was detected as we
registered a picture of his in the previous tutorial.
Note also that the full response contains the coordinates of the faces.


Extracting Faces
----------------

The face coordinates allows you to easily extract the detected faces.
Here we shall use `Image Sharp <https://github.com/SixLabors/ImageSharp />`_ to extract the faces and save them ::

    using System;
    using System.IO;
    using System.Net.Http;
    using System.Threading.Tasks;
    using Newtonsoft.Json;
    using SixLabors.ImageSharp;
    using SixLabors.ImageSharp.Processing;
    using SixLabors.Primitives;

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

            var width = user.x_max - user.x_min;
            var height = user.y_max - user.y_min;

            var crop_region = new Rectangle(user.x_min,user.y_min,width,height);

            using(var image = Image.Load(image_path)){

                image.Mutate(x => x
                .Crop(crop_region)
                );
                image.Save(user.userid + ".jpg");

            }

        }

        }

        static void Main(string[] args){

            recognizeFace("test-image2.jpg").Wait();

        }

    }

    }
    
Result

    .. figure:: Idris-Elba.jpg
        :align: center

    .. figure:: unknown.jpg
        :align: center

**Setting Minimum Confidence**

DeepStack recognizes faces by computing the similarity between the embedding of a new face and the set of embeddings of previously registered faces.
By default, the minimum confidence is 0.67. The confidence ranges between 0 and 1.
If the similarity for a new face falls below the min_confidence, unknown will be returned.
The min_confidence parameter allows you to increase or reduce the minimum confidence.

We lower the confidence allowed below.

Example ::

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
        request.Add(new StringContent("0.30"),"min_confidence");
        var output = await client.PostAsync("http://localhost:80/v1/vision/face/recognize",request);
        var jsonString = await output.Content.ReadAsStringAsync();
        Response response = JsonConvert.DeserializeObject<Response>(jsonString);

        foreach (var user in response.predictions){

            Console.WriteLine(user.userid);

        }

        Console.WriteLine(jsonString);

    }

    static void Main(string[] args){

        recognizeFace("test-image2.jpg").Wait();

    }

    }

    }

Result ::

    Idris Elba
    Adele
    Full Response:  {'success': True, 'predictions': [{'userid': 'Idris Elba', 'y_min': 154, 'x_min': 1615, 'x_max': 1983, 'confidence': 0.76965684, 'y_max': 682}, {'userid': 'Adele', 'y_min': 237, 'x_min': 869, 'x_max': 1214, 'confidence': 0.6044803, 'y_max': 732}]}

By reducing the allowed confidence, the system detects the first face as Adele. The lower the confidence, the more likely
for the system to make mistakes. When the confidence level is high, mistakes are extremely rare, however, the system may 
return unknown always if the confidence is too high.

**For security related processes such as authentication, set the min_confidence at 0.7 or higher**

Managing Registered Faces
--------------------------

The face recognition API allows you to retrieve and delete faces
that has been previously registered with DeepStack.

Listing faces ::

    using System;
    using System.IO;
    using System.Net.Http;
    using System.Threading.Tasks;

    namespace app
    {

    class App {

        static HttpClient client = new HttpClient();

        public static async Task listFaces(){

            var output = await client.PostAsync("http://localhost:80/v1/vision/face/list",null);
            var jsonString = await output.Content.ReadAsStringAsync();
            
            Console.WriteLine(jsonString);

        }

        static void Main(string[] args){

            listFaces().Wait();
            
        }

    }
    
    }

Result ::

    {'success': True, 'faces': ['Tom Cruise', 'Adele', 'Idris Elba', 'Christina Perri']}


Deleting a face ::

    using System;
    using System.IO;
    using System.Net.Http;
    using System.Threading.Tasks;

    namespace app
    {

    class App {



        static HttpClient client = new HttpClient();

        public static async Task registerFace(string userid){

            var request = new MultipartFormDataContent();
            request.Add(new StringContent(userid),"userid");
            var output = await client.PostAsync("http://localhost:80/v1/vision/face/delete",request);
            var jsonString = await output.Content.ReadAsStringAsync();
            
            Console.WriteLine(jsonString);

        }

        static void Main(string[] args){

            registerFace("Idris Elba").Wait();
            
        }

    }
    
    }

Result ::

    {'success': True}

**Performance**

DeepStack offers three modes allowing you to tradeoff speed for peformance. 
During startup, you can specify performance mode to be , **"High" , "Medium" and "Low"**

The default mode is "Medium"

You can speciy a different mode as seen below ::

    sudo docker run -e MODE=High -e VISION-FACE=True -v localstorage:/datastore \
    -p 80:5000 deepquestai/deepstack

Note the -**e MODE=High** above 