.. DeepStack documentation master file, created by
   sphinx-quickstart on Wed Dec 12 17:30:35 2018.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Face Detection
==============

The face detection API detects faces and returns their coordinates as well as the gender.
It functions similarly to the face recognition API except that it does not 
perform recognition. 
Also note that the recognition API does not return gender predictions.

**Example**

.. figure:: family.jpg
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

        public string gender {get;set;}
        public float confidence {get;set;}
        public int y_min {get;set;}
        public int x_min {get;set;}
        public int y_max {get;set;}
        public int x_max {get;set;}
    
    }

    class App {

        static HttpClient client = new HttpClient();

        public static async Task detectFace(string image_path){

            var request = new MultipartFormDataContent();
            var image_data = File.OpenRead(image_path);
            request.Add(new StreamContent(image_data),"image",Path.GetFileName(image_path));
            var output = await client.PostAsync("http://localhost:80/v1/vision/face",request);
            var jsonString = await output.Content.ReadAsStringAsync();
            Response response = JsonConvert.DeserializeObject<Response>(jsonString);
            
            foreach (var user in response.predictions){

                Console.WriteLine(user.gender);

            }

            Console.WriteLine(jsonString);

        }

        static void Main(string[] args){

            detectFace("family.jpg").Wait();

        }

    }
    
    }

Result ::

    female
    male
    male
    female
    {'predictions': [{'y_max': 303, 'gender': 'female', 'confidence': 0.99999213, 'x_min': 534, 'x_max': 629, 'y_min': 174}, {'y_max': 275, 'gender': 'male', 'confidence': 0.6611953, 'x_min': 616, 'x_max': 711, 'y_min': 146}, {'y_max': 259, 'gender': 'male', 'confidence': 0.99884146, 'x_min': 729, 'x_max': 811, 'y_min': 147}, {'y_max': 290, 'gender': 'female', 'confidence': 0.99997365, 'x_min': 471, 'x_max': 549, 'y_min': 190}], 'success': True}

We can use the coordinates returned to extract the faces from the image

::

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

    public string gender {get;set;}
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
        var output = await client.PostAsync("http://localhost:80/v1/vision/face",request);
        var jsonString = await output.Content.ReadAsStringAsync();
        Response response = JsonConvert.DeserializeObject<Response>(jsonString);

        var i = 0;

        foreach (var user in response.predictions){

            var width = user.x_max - user.x_min;
            var height = user.y_max - user.y_min;

            var crop_region = new Rectangle(user.x_min,user.y_min,width,height);

            using(var image = Image.Load(image_path)){

                image.Mutate(x => x
                .Crop(crop_region)
                );
                image.Save(user.gender + i.ToString() + "_.jpg");

            }

            i++;

        }

        }

        static void Main(string[] args){

            recognizeFace("family.jpg").Wait();

        }

    }

    }

Result

.. figure:: image0_female.jpg
    :align: center

.. figure:: image1_male.jpg
    :align: center

.. figure:: image2_male.jpg
    :align: center

.. figure:: image3_female.jpg
    :align: center