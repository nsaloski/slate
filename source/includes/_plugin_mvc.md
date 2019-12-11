## MVC

Caymland uses a Model-View-Controller structure to manage user interaction on the frontend (views) with the underlying code processes (controllers and models). ([Entity and Repository classes](#database) are also used to manage interaction with the database).

In Symfony, and thus Caymland, the controller is the center of the MVC structure. The route requested by the user determines the controller method to call. The controller will then communicate with the model to get or manipulate data then display it to the user through the view.


             
            
               
               


