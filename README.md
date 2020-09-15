# Authentication

## Table of Contents  
* [Goal](#Goal)<br>
* [How Authentication Works ?](#How-Authentication-Works-?)<br>
* [Diagram Showing Web Token Based Authentication](#Diagram-Showing-Web-Token-Based-Authentication)<br>
* [Adding Authentication Section into Project](#Adding-Authentication-Section-into-Project)<br>
* [Switching between Authentication Modes](#Switching-between-Authentication-Modes)<br>
* [Handling Form Inputs ](#Handling-Form-Inputs)<br>
* [Firebase Setup](#Firebase-Setup)<br>
* [Preparing Signup Request](#Preparing-Signup-Request)<br>
* [Sending a Signup Request](#Sending-a-Signup-Request)<br>
* [Adding Loading Spinner](#Adding-Loading-Spinner)<br>
* [Handling an Error only for Signup Request](#Handling-an-Error-only-for-Signup-Request)<br>
* [Sending Login Request](#Sending-Login-Request)<br>
* [Handling The Error From Sign in as well as Signup Request](#Handling-The-Error-From-Sign-in-as-well-as-Signup-Request)<br>
* [Creating and Storing User Data](#Creating-and-Storing-User-Data)<br>
* [Reflecting Auth State in UI](#Reflecting-Auth-State-in-UI)<br>
* [Make Fetching Work Again](#Make-Fetching-Work-Again)<br>
* [Make Save Data Work Again Using Interceptor](#Make-Save-Data-Work-Again-Using-Interceptor)<br>
* [Adding Functionality to Logout](#Adding-Functionality-to-Logout)<br>
* [Add Auto Login](#Add-Auto-Login)<br>
* [Add Auto Logout](#Add-Auto-Logout)<br>
* [Adding Auth Guard](#Adding-Auth-Guard)<br>

## Goal

1. Create an authentication page with two input fields
2. input 1: Email
3. input 2: Password
4. two buttons [Sign up | Login]

## How Authentication Works ?

1. In Conventional approach Authentication in **multiple page** web application is performed using session.
2. In Angular based single page application, where communication between client and server happens using REST API
3. And the authentication in Angular app happens over **JSON web token**
4. When Angular client sends authentication request to server, server responses with web token if authentication credentials are valid
5. Web token includes ENCODED string with meta data
6. Web token does not include the encrypted string

## Diagram Showing Web Token Based Authentication

![auth diagram](assets/authDiagram.PNG)

## Adding Authentication Section into Project

1. Create the component which holds the authentication logic and templet

   ~~~typescript
   app
    |---Auth
          |---- auth.component.ts [logic]
          |---- auth.component.html [templet]
   ~~~

2. Basic skeleton of ```auth.component.ts```

   ~~~typescript
   import { Component } from "@angular/core";
   
   @Component({
       selector: 'app-auth',
       templateUrl: './auth.component.html',
   })
   export class AuthComponent{
   }
   ~~~

3. An authentication templet ```auth.component.html```

   ~~~html
   <div class="row">
       <div class="col-xs-12 col-md-6 col-md-offset">
           <form>
               <div class="form-group">
                   <label for="email">E-mail</label>
                   <input type="text" id="email" class="form-control">
               </div>
   
               <div class="form-group">
                   <label for="password">Password</label>
                   <input type="password" id="password" class="form-control">
               </div>
   
               <div>
                   <button class="btn btn-primary"> Sign up</button> | 
                   <button class="btn btn-primary"> Switch to Login</button>
               </div>
           </form>
       </div>
   </div>
   ~~~

4. Registering the auth component in ```app.module.ts```

   ~~~typescript
   @NgModule({
       declarations: [ AuthComponent ]
   })
   ~~~

5. Setup routing for Authentication page in ```app-routing.module.ts```

   ~~~typescript
   const appRoutes: Routes = [
       {path: 'auth', component: AuthComponent}
       ]
   ~~~

6. App Authentication tab in header ```header.component.html```

   ~~~html
   <div class = "navbar-collapse">
       <ul>
           <li>Recipe</li>
           <li>Shopping list</li>
           
            <li routerLinkActive = "active">
               <a routerLink = "/auth">Authentication</a>
            </li>
       </ul>
   </div>
   ~~~

## Switching between Authentication Modes

1. Switch between Login / Sign in mode

2. Adding mode property and method which change the property in ```auth.component.ts```

   ~~~typescript
   export class AuthComponent{
       isLoginMode = true;		// mode property
       onSwitchMode(){			// method which can change the mode property
           this.isLoginMode = !this.isLoginMode;
       }
   }
   ~~~

3. Accessing the above implemented property from templet ```auth.component.html```

   usage of ternary operation to check the mode and choose the button text

   ~~~html
   
   <!--The input fields will remain same for both login and signup-->
   <div>
     <button class="btn btn-primary" 
             type="submit"> 
           {{isLoginMode ? 'Login' : 'Signup'}}
     </button> | 
   
     <button class="btn btn-primary" 
             (click) = "onSwitchMode()" 
             type="button"> 
           Switch to {{isLoginMode ?  'SignUp' : 'Login'}}
     </button>
   </div>
   ~~~

## Handling Form Inputs 

1. Add tags on the input fields

   ~~~html
    <input type="text" 
           name = "email"
           ngModel
           required
           email
    >
   
    <input type="password" 
           name = "password"
           ngModel
           required
           minlength="6"
    >
   
   <!--name and ngModel are tags to register the control in form-->
   <!--required, email and minlength are validators-->
   ~~~

2. Declare local reference of the form and use it as a parameter to the submit method

   ~~~html
    <form (ngSubmit)="onSubmit(authForm)" #authForm = ngForm>
        ....
   </form>  
   <!--#authForm is the local reference of the form-->
   <!--onSubmit() get executed on click of submit button-->
   <!--authForm a reference is passed as a parameter to the onSubmit method-->
   ~~~

3. Implement ```onSubmit()``` method in ```auth.component.ts```

   ~~~typescript
   onSubmit(form: NgForm){
           console.log(form.value);	// logging the obtained form values
           form.reset();			   // resetting the form 
   }
   ~~~

## Firebase Setup

1. Set the database rules

   ![databaseSetup](assets/dbsetup.PNG)

2. Setup the authentication

   ![authSetup](assets/authentication1.PNG)

   ![auth2](assets/authentication2.PNG)

## Preparing Signup Request

1. Create ```auth.service.ts``` in auth folder

   This service will be responsible for user signing up, signing in and managing the web  token

2. Basic skeleton of the auth service

   ~~~typescript
   @Injectable({
       providedIn: 'root'
   })
   export class AuthService{
       // following url is signup url and api key is obtained from project setting
       private signUpURL = "link obtained from firebase auth api signup + API key of project"
       
       constructor(private http : HttpClient){
       }
       
       signUp(email:string, password:string){
       }
   ~~~

3. Implement the signup method in ```auth.service.ts```

   ~~~typescript
   // this method returns the observable
   signUp(email:string, password:string){
         return this.http.post(this.signUpURL,
                 {email:email, 			
                 password:password,
                 returnSecureToken:true
                 })
   }
   // firebase api needs email, password and returnsSecureToken with signup request
   // email and password will be obtained as the method parameters
   // subscription to the post request will be done in the method which calls this method
   ~~~

4. Implement Interface ```AuthResponseData{ }```

   ~~~typescript
   // firebase gives following object as the response for signup post request
   // can be found in the firebase api doc 
   interface AuthResponseData {
       idToken : string;
       email : string;
       refreshToken : string;
       expiresIn : string;
       localId : string;
   }
   
   @Injectable({ providedIn: 'root' })
   export class AuthService{ 
   }
   ~~~

5. Add specific response to the generic post request as follow

   ~~~typescript
   // <AuthResponseData> included in the post method
   // by default post request is generic
   // by adding <AuthResponseData> we are specifying the response of the post request
   signUp(email:string, password:string){
       return this.http.post<AuthResponseData>(this.signUpURL,
               {email:email, 
               password:password,
               returnSecureToken:true
               })
   }
   ~~~

## Sending a Signup Request

1. Inject the ```AuthService``` into ```auth.component.ts```

   ~~~typescript
   export class AuthComponent{
       constructor(private authSer:AuthService){}
   }
   ~~~

2. Obtain the username and password in ```onSubmit(form:NgForm)``` method of ```app.component.ts```

   ~~~typescript
    onSubmit(form: NgForm){
           const email = form.value.email;
           const password = form.value.password;
    }
   ~~~

3. Check if ```isLogingMode``` property is set?

   ~~~typescript
    onSubmit(form: NgForm){
           const email = form.value.email;
           const password = form.value.password;
      
        	if (this.isLoginMode) {
       		// signin logic
           } 
        	else {
           // sign up logic implemented in next point
           }
    }	
   ~~~

4. call the ```signUp()``` method of ```auth.service.ts``` and subscribe to it

   ```typescript
   // following code will be reside in else section of above point
   this.authSer.signUp(email, password).subscribe(
              responseData => {				// response of the request obtained as an arrow func
                   console.log(responseData);	 // logging the obtained responseData
              },
              error => {					   // error response obtain through arrow function
                   console.log(error);			// logging the obtained error response
              }
    );
   ```

## Adding Loading Spinner 

1. Loading spinner will be appear when app is performing request to the firebase

2. Loading spinner source: https://loading.io/css/

3. create loading-spinner component

   ~~~typescript
   shared
      |---- loading-spinner
   			   |-------- loading-spinner.css
   			   |-------- loading-spinner.ts
   ~~~

4. Content of ```loading-spinner.ts```

   ~~~typescript
   import { Component } from "@angular/core";
   
   @Component({
       selector: 'app-loading-spinner',
       template: '<div class="lds-facebook"><div></div><div></div><div></div></div>',
       styleUrls: ['./loading-spinner.css']  
   })
   export class LoadingSpinnerComponent{}
   ~~~

5. Register loading spinner component in the ```app.module.ts```

   ~~~stylus
   import { LoadingSpinnerComponent } from './shared/loading-spinner/loading-spinner';
   
   @NgModule({
       declarations: [LoadingSpinnerComponent]
       })
   ~~~

6. Add new property ```isLoading : boolean``` in ```auth.component.ts``` 

   ~~~typescript
   export class AuthComponent{
       isLoading = false;
       
       onSubmit(form: NgForm){
           const email = form.value.email;
           const password = form.value.password;
           this.isLoading = true;		// loading started
           if (this.isLoginMode) { // signin logic
           } else { // sign up logic
               this.authSer.signUp(email, password).subscribe(
                   responseData => {
                       console.log(responseData);
                       this.isLoading = false;	// loading ends 
                   },
                   error => {
                       console.log(error);
                       this.isLoading = false;	// loading ends
                   }
               );
           }
      		form.reset();
       }
   }
   ~~~

7. Add the loading-spinner component in ```auth.component.html```

   ~~~html
   <div class="row">
       <div class="col-xs-12 col-md-6 col-md-offset">
   
           <div *ngIf = "isLoading" style="text-align: center;">
               <app-loading-spinner></app-loading-spinner>
           </div>
           
           <form *ngIf = "!isLoading">
               <!--load the form if loading is false-->
           </form>
       </div>
   </div>    
   ~~~

## Handling an Error only for Signup Request

1. Declaring property name ```error : string``` in the ```auth.component.ts```

   ~~~typescript
   export class AuthComponent{
   	error : string = null;
   }
   ~~~

2. Bind the aforementioned property with ```auth.component.html``` templet

   ~~~html
   <div class="row">
       <div class="col-xs-12 col-md-6 col-md-offset">
   		<div  class="alert alert-danger" *ngIf = "error">
       		<p> {{ error }}</p>
   		</div>
       </div>
   </div>
   ~~~

3. Extract the error message in ```auth.service.ts```

   ~~~typescript
   // pipe rxjs operator is applied to the post request of signup method
   .pipe(catchError(
       errorRes => {
        		let errorMessage = 'An unknown error occured';	// default error message
       
         		if(!errorRes.error || !errorRes.error.error){
                 		return throwError(errorMessage);	   // if unknown error occures throw this
         		}
          		switch (errorRes.error.error.message){
                 		case 'EMAIL_EXISTS': 				  // if this error occures
                 			errorMessage = 'This email exists already'	// change the errorMessage
          		}
       
           	return throwError(errorMessage);		// throw error here if known error occured
   	}
   ));
   ~~~

4. obtain the error message in ```auth.component.ts```

   ```typescript
   this.authSer.signUp(email, password).subscribe(
            responseData => {
                  console.log(responseData);
                  this.isLoading = false;
            },
            errorMessage => {
                // assign errorMessage to the previously declared error property
                  this.error = errorMessage;	
                  this.isLoading = false;
            }
   );
   ```

## Sending Login Request

1. Obtain URL for Login from firebase API doc

2. create ```Login(email:string, password:string)``` in ```auth.service.ts```

3. perform post request in login method

   ~~~typescript
   // only prepare the post observable but no subscribe here return the observable
   // post request needs email password and returnSecureToken as data
   login(email:string, password:string){   
       return this.http.post<AuthResponseData>(this.loginURL,
                   {email:email,
                   password:password,
                   returnSecureToken: true   
                   });
   }
   ~~~

4. Add ```registered``` as an optional property in ```AuthResponseData``` interface

   ~~~typescript
   // registered property is yield by Signin request but
   // this property is not yield by Signup request 
   // check firebase API doc for list of properties in response data
   interface AuthResponseData {
       idToken : string;
       email : string;
       refreshToken : string;
       expiresIn : string;
       localId : string;
       registered? : Boolean	// optional property
   }
   ~~~

5. modify the ```AuthResponseData``` interface in ```auth.service.ts```

   ~~~typescript
   // make it export
   export interface AuthResponseData {}
   ~~~

6. separate the subscription code from login and sign up mode

   ~~~typescript
   // import the Authresponse data from auth.service.ts
   import { AuthService, AuthResponseData } from "./auth.service";
   
   // create the observable variable as follow
   let authObservable : Observable<AuthResponseData>;
   
   // subscribe to this observable in onSubmit method outside if else block
   onSubmit(form: NgForm){
           const email = form.value.email;
           const password = form.value.password;
       
           // following variable will hold the observable
           let authObservable : Observable<AuthResponseData>;
           this.isLoading = true;
   
           if (this.isLoginMode) {
               // signin logic
               // assigning the login observable to the authObservable variable
               authObservable = this.authSer.login(email, password);
           } else {
               // sign up logic
               // assigning signup observable to the authObservable variable
               authObservable = this.authSer.signUp(email, password);
           }
       
   		// execute the authObservable here after
       	// appropreate assignment of observable
           authObservable.subscribe(
               responseData => {
                   console.log(responseData);
                   this.isLoading = false;
               },
               errorMessage => {
                   this.error = errorMessage;
                   this.isLoading = false;
               }
           );
           form.reset();
   }
   ~~~

## Handling The Error From Sign in as well as Signup Request

1. Centralizing the error handling by implementing ```handleError()``` method in ```auth.service.ts```

   ~~~typescript
   private handleError(errorResponse : HttpErrorResponse){
       
       // errorMessage property will hold the error message obtained from request
       let errorMessage = 'An unknown error occured';	// default message
       
       // check if errorResponse is empty?
       if(!errorResponse.error || !errorResponse.error.error){
               return throwError(errorMessage);	// then send the default error message
           }
       
       // if errorResponse is not empty then go to the switch case
       switch (errorResponse.error.error.message){
               // sign up related errors
                   case 'EMAIL_EXISTS': 	// if errorResponse is this then
                   errorMessage = 'This email exists already'; // put this value in errorMessage
                   break;
               
               	case 'TOO_MANY_ATTEMPTS_TRY_LATER':	
                   errorMessage = 'Too many attempts try again later'; 
                   break;
               
       		// sign in related errors   
                   case 'INVALID_PASSWORD':
                   errorMessage = 'Invalid Password';
                   break;
               
                   case 'EMAIL_NOT_FOUND':
                   errorMessage = 'This email does not found';
                   break;
       }
       return throwError(errorMessage); // throw error message before leaving this method
   }
   ~~~

2. Modify the ```signUp``` and ```login``` methods of ```auth.service.ts```

   ~~~typescript
   signUp(email:string, password:string){
           return this.http.post<AuthResponseData>(this.signUpURL...)
                   .pipe(catchError(this.handleError));	// calling the handleError method
       }
   
   login(email:string, password:string){ 
           return this.http.post<AuthResponseData>(this.loginURL...)
                   .pipe(catchError(this.handleError));	// calling the handleError method
       }
   
   // this error eventually will end in authObservable subscription block of auth.component.ts
   ~~~

   ![errorHandlingDiagram](assets/errorhandling.PNG)

3. ```auth.component.ts``` is untouched

## Creating and Storing User Data

1. Create the user model

   ~~~typescript
   // constructor automatically create the instance of property and assign it to it 
   export class User {
       constructor(public email:string, 
           public id : string, 
           private _token: string, 
           private _tokenExpirationDate : Date){} 
   }
   ~~~

2. Add get token method in the model

   ~~~typescript
   get token(){
       	// if token expired then return null
           if(!this._tokenExpirationDate || new Date() > this._tokenExpirationDate){
               return null;
           }
       	// or return the token 
           return this._token;
       }    
   ~~~

3. Create the user subject in the ```auth.service.ts```

   ~~~typescript
   userx = new Subject<User>();
   ~~~

4. create the ```userAuthentication``` method

   ~~~typescript
   private userAuthentication(email:string, userId:string, token: string, expiresIn: number){
         const expirationDate = new Date(
               new Date().getTime() + expiresIn * 1000);
   
         const user = new User(email, 
                                 userId,
                                 token,
                                 expirationDate);
   
         this.user.next(user);
   }
   ~~~

5. execute the above implemented method in signup and login under pipe under tap

   ~~~typescript
   // we can tap the response data from post request inside pipe
   // to create the user object userAuthentication method is used
   signUp(email:string, password:string){
        return this.http.post<AuthResponseData>(this.signUpURL...)
       		.pipe(
            		catchError(this.handleError)
            		,tap( resData => {
                           this.userAuthentication(resData.email, resData.localId, 									  resData.idToken, +resData.expiresIn);
                   	}
                 ));       
   }
                    
   login(email:string, password:string){   // only prepare the observable but no subscribe here
          return this.http.post<AuthResponseData>(this.loginURL...)
                 .pipe(
            		catchError(this.handleError)
                    ,tap( resData => {
                            this.userAuthentication(resData.email, resData.localId, 
                            resData.idToken, +resData.expiresIn);
                        }
                  ));
       }                     
   ~~~

## Reflecting Auth State in UI

1. Inject router in the ```auth.component.ts```

   ~~~typescript
       constructor(private authSer:AuthService, 
                   	private router:Router){}
   ~~~

2. Redirect to the recipe component upon successful login

   ~~~typescript
   // perform this task in authObservable's subscribe method
    onSubmit(form: NgForm){
        ...
        authObservable.subscribe(
           responseData => {
               console.log(responseData);
               this.isLoading = false;
               this.router.navigate(['/recipes']);	// navigate to the recipe component
           },
           errorMessage => {
               this.error = errorMessage;
               console.log(errorMessage);
               this.isLoading = false;
               }
           );
    }
   ~~~

3. Inject ```auth.service.ts``` in ```header.component.ts``` and declare ```isAuthenticated``` property

   ~~~typescript
   // this property will be used in html to control the header
   isAuthenticated = false;
   
   constructor(private remote:DataStorageService, 
                   private authService : AuthService){
       }
   ~~~

4. Subscribe to the ```user``` subject of ```AuthService``` and at the end unsubscribe

   ~~~typescript
   private userSub : Subscription; // a property which holds the subscription
   
   ngOnInit(){
       this.userSub = this.authService.user.subscribe(
           user => {
               // is authenticated = true if we receive user object from subscription
               // is authenticated = false if we dont receive user object[null] from subscription 
               this.isAuthenticated = !user ? false :true;
           }
       );
   }
   
   ngOnDestroy(){
       this.userSub.unsubscribe();
   }
   ~~~

5. Update the ```header.component.html``` component

   ~~~html
   <div class="navbar-collapse">
       
        <ul class = "nav navbar-nav">
            <li *ngIf = "isAuthenticated">
                <a>Recipes</a>
            </li>
            <li>
            	<a>Shopping List</a>
            </li>
            <li *ngIf = "!isAuthenticated">
                <a>Authentication</a>
            </li>  
       </ul>
       
       <ul class="nav  navbar-nav navbar-right">
           <li *ngIf = "isAuthenticated">
               <a>Logout</a>
           </li>
           <li class="dropdown" *ngIf = "isAuthenticated">
               <a>Manage</a>
           </li>
       </ul>
       
   </div>
   ~~~

## Make Fetching Work Again

1. Attaching token to the outgoing request [fetch]

2. Inject ```authService``` in ```data-storage.service.ts``` home of ```storeRecipe()``` and ```fetchRecipe()``` methods

   ~~~typescript
   constructor(private http: HttpClient, 
           private authService : AuthService){
       }
   ~~~

3. Change the ```user``` subject in ```auth.service.ts```

   ~~~typescript
   // we can now user event after it got emitted [at any time]
   user = new BehaviorSubject<User>(null);
   ~~~

4. Modify the ```fetchRecipe()``` of ```data-storage.service.ts``` [exhaustMap]

   ~~~typescript
   fetchRecipes(){
       return this.authService.user.pipe(take(1),
                   exhaustMap(user => { // it will subscribe the user get the data and unsubscribe it
                       return this.http
                           .get<Recipe[]>(this.remoteUrl,
                               {
                                   params: new HttpParams().set('auth', user.token)
                               }
                               );
                   })
                   ,map(recipes =>{               
                       return recipes.map(recipe => {
                           return {...recipe, ingredients: recipe.ingredients ? recipe.ingredients:[]
                           };
                       });           
                   })
                   ,tap(recipes => {
                       this.recipeService.setRecipes(recipes);
                    })
       )
   }
   ~~~

## Make Save Data Work Again Using Interceptor

1. Create ```auth-interceptor.ts``` in ```auth``` folder

   ~~~typescript
   @Injectable()
   export class AuthIntercepterService implements HttpInterceptor{
        constructor(private authService: AuthService){} // injecting authService dependency
       
       // every class which implements HttpInterceptor must have following method  in it
       intercept(){}
   }
   ~~~

2. Implement ```Intercept()``` [Intercepter is executed before the request is being made]

   ~~~typescript
   // param1 : HttpRequest [req]
   // param2 : Httphandler [next]
   intercept(req:HttpRequest<any>, next: HttpHandler){
       // will return the modified request 
       // this modified request will have user credentials 
       // idea is to authenticate the user before peroforming any post or get req.
   }
   ~~~

3. ```Intercept()``` code

   ~~~typescript
   intercept(req:HttpRequest<any>, next: HttpHandler){
        return this.authService.user.pipe(take(1),
        exhaustMap(user => {
            // if there is no user then 
            // simply return the req. obtained from paramter of this intercept method1
             if(!user){
                 return next.handle(req);
             }
            // make the clone of inputed req. 
            // then add the Http authentication param to that cloned req. 
             const modifiedReq = req.clone({
                 params: new HttpParams().set('auth',user.token)
             });
            // return the modified request which has actual req. as well as user auth
             return next.handle(modifiedReq);
         }));
   }
   ~~~

4. Provide this interceptor in ```app.module.ts```

   ~~~typescript
   @NgModule({
       providers: [{
           provide: HTTP_INTERCEPTORS,
           useClass: AuthIntercepterService,
           multi: true
       }]
   })
   ~~~

5. Modify the ```data-storage.service.ts``` specifically ```fetchRecipes()```

   ~~~typescript
   fetchRecipes(){
       return this.http.get<Recipe[]>(this.remoteUrl).pipe(
           map(recipes =>{               // this is rxjs map operator
               return recipes.map(recipe => {
                     return {...recipe, ingredients: recipe.ingredients ? recipe.ingredients:[]};
                           }
                      );           // this map is javascript array method
               }
           )
           ,tap(recipes => {
                    this.recipeService.setRecipes(recipes);
                  }
            )
        )
   }
   // storeRecipes() will remain same we are just itroducing the itercept so previously modified 
   // fetch method will be remodified
   ~~~

## Adding Functionality to Logout

1. Implement ```logout()``` in ```auth.service.ts```

   ~~~typescript
   logout(){
      this.user.next(null);   // make the user object null
   }
   // user object is situated in the same service
   ~~~

2. Call this above implemented logout method in ```header.component.ts```

   ~~~typescript
   onLogout(){
      this.authService.logout();
   }
   ~~~

3. Add click listener to above implemented method in ```header.component.ts```

   ~~~html
   <li  *ngIf = "isAuthenticated">
          <a style = "cursor:pointer;" (click) = "onLogout()">Logout</a>
   </li>
   ~~~

4. Perform redirection of the user when logout is clicked

   ~~~typescript
   // redirection will be performed in logout() of the auth.service.ts 
   // because logout will be performed from different locations
   logout(){
        this.user.next(null);   // make the user object null
        this.router.navigate(['/auth']);
   }
   ~~~

## Add Auto Login

1. When page reload happens, application restarts and old user credentials are wiped out

2. Whenever we reload the page we need to login again to access the app

3. In this section an auto login will be implemented 

4. So that even after restart user remains logged in

5. User can logout using Logout button or when user token get expired

6. Idea is to store the user data into persistent storage [local storage]

7. Store the logged user in the local storage as follow

   ~~~typescript 
   // this method is already implemented in previous sections
   private userAuthentication(){
       localStorage.setItem('userData', JSON.stringify(user));
   }
   // convert the user object into plain text using JSON.stringify() method
   ~~~

8. Implement ```autoLogin()``` in ```auth.service.ts```

   ~~~typescript
   autoLogin(){
       // obtain the user data from local storage
       const userData:{
               email:string;
               id:string;
               _token:string;
               _tokenExpirationDate:string;
       } = JSON.parse(localStorage.getItem('userData')); 
       // JSON.parse will converte text into object
       // userData will hold the user which is stored in the local storage
       
       // if there is no user data available in local storage
       // then dont perform autoLogin
       if(!userData){
           return;
       } 
       
       // create the object of user which is already in local storage
       const LoadedUser  = new User(userData.email, 
                                    userData.id, 
                                    userData._token, 
                                    new Date(userData._tokenExpirationDate)
       );
       
       // check if loaded user has valid token
       // if yes then publish loaded user over the user subject
       // user subject is implemented in the same service
       if(LoadedUser.token){
             this.user.next(LoadedUser);  
       }
   }
   ~~~

9. Add ```autoLogin()``` into the ```ngOnInit``` hook of ```app.component.ts```

   ~~~typescript
   export class AppComponent implements OnInit {
     	constructor(private authSer: AuthService){}
     	ngOnInit(){
         this.authSer.autoLogin();
     	}
   }
   ~~~

## Add Auto Logout

1. Even after log out click, we can still logged in by refreshing the application

2. Because user data is stored in local storage

3. This user data does not delete even after refreshing the page 

4. And we directly do the login even without authentication upon reloading

5. Also when user token expires, we are still logged in

6. Clear the user data upon the execution of ```logout()``` method of ```auth.service.ts```

   ~~~typescript
   logout(){
       // clear the user data locally stored
       localStorage.removeItem('userData');
       
       // follow section 7 for this
        if(this.tokenExpirationTimer){
               clearTimeout(this.tokenExpirationTimer);
           }
           this.tokenExpirationTimer = null;
   }
   ~~~

7. Implement ```autoLogout()``` in ```auth.service.ts```

   ~~~typescript
   // create the timer expiration property 
   // this property will be used to check whether token timeout happened or not
   private tokenExpirationTimer: any;
   
   // in autoLogout() we will set the above declared timer 
   // and once that timer expires perform logout method
   autoLogout(expirationDuration : number){
         this.tokenExpirationTimer = setTimeout(() =>{
               							this.logout();	
           									},expirationDuration);
   }
   ~~~

8. Call the ```autoLogout()``` method in ```userAuthentication()``` of the ```auth.service.ts```

   ~~~typescript
    private userAuthentication(......expiresIn: number){
        this.user.next(user);
        this.autoLogout(expiresIn*1000);	// autologout in [expiresIn*1000] duration
        localStorage.setItem('userData', JSON.stringify(user));
    }
   ~~~

9. Call ```autoLogout()``` in ```autoLogin()``` method of same service

   ~~~typescript
    autoLogin(){
        if(LoadedUser.token){
         this.user.next(LoadedUser);  
         const expirationDuration = new Date(userData._tokenExpirationDate)
         							  .getTime() - new Date().getTime();
         this.autoLogout(expirationDuration);
         }
    }
   
   // basically when we calls autoLogout method, we are just setting the timer and when this timer overflow logout method is called 
   // the value for timer is passed as a parameter to the autologin method
   ~~~

## Adding Auth Guard

1. The recipes tab is not visible unless user is logged in 

2. Although user can navigate to the recipes component via URL 

3. Bug: user can navigate to the recipes component without any authentication via URL

4. To avoid user to navigate to recipes section without authentication a guard need to be implemented

5. Create ```auth.guard.ts ```under auth folder

   ~~~typescript
   @Injectable({
       providedIn: 'root'})
   export class AuthGuard implements CanActivate {
       // import authService and Router
       constructor(private authService: AuthService,
       private router: Router) { }
       
      //canActivate method
   }
   ~~~

6. Implement ```canActivate()```

   ~~~typescript
   canActivate(route: ActivatedRouteSnapshot, router: RouterStateSnapshot):
   // following things can be returned
        | boolean
        | UrlTree
        | Promise<boolean | UrlTree>
        | Observable<boolean | UrlTree> {
        return this.authService.user.pipe(take(1),map(user => {
            const isAuth = !!user;	
            if (isAuth) {
               return true;
            }
            return this.router.createUrlTree(['/auth']);
        }));
   }
   ~~~

7. Apply guard on the recipe route in ```app-routing.module.ts```

   ~~~typescript
   const appRoutes: Routes =  [
       {   path: 'recipes', component: RecipesComponent, 
           canActivate: [AuthGuard],
           children: [...]
       }    
   ]
   ~~~

8. Take(1) in ```canActivate()``` method of ```Auth.guard.ts```































