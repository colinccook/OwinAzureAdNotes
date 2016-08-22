### OWIN ENABLING A WEB PROJECT

 - Create a new ASP.NET MVC project with Windows Authentication
   (this is an example of an existing web projet)
 - Turn off Windows Authentication and turn on Annonymous Authentication
 - Put in _Layout.cshtml: <p>User is authenticated: @User.Identity.IsAuthenticated</p>

Test it works? You may also need to adjust web.config:

<authorization>
      <deny users="?" />
      <allow users="domain\User" />
</authorization>

 - Install-Package Microsoft.Owin
 - Install-Package Microsoft.Owin.Host.SystemWeb
 - Add OWIN Startup file to project
 - Implement a piece of OWIN middleware inline

            app.Use((IOwinContext context, Func<Task> next) =>
            {
                context.Response.Write("Hello world");
                // context.Response.Write(context.Request.Path.Value);
                return next.Invoke(); // continue to the next owin middleware
 		// return Task.FromResult(0); // stop the pipeline
            });

 - After testing, enable SSL and make a note of SSL URL (https://localhost:44310/)

SETTING UP AN EXTERNAL AUTHENTICATION SOURCE (we're using Azure Active Directory)
=================================================================================

OWIN middlewares for Facebook, Twitter, Google, Microsoft and 
Azure Active Directory (Office 365) exists!!

We will create a test Azure Active Directory with a User to login with! (AADs are free)

 - Log into Azure Portal with your Microsoft personal account
 - Go to Active Directory service, opens "old Portal"
 - Create a new Azure Active Directory (Firstname Lastname Owin Demo), make a note of domain name
	(fact, the CCN AAD is logon@studentccnac.onmicrosoft.com!)
 - Add a User. Make a note of Username (Bob.Smith@ColinCookOwinDemo.onmicrosoft.com)
   and password (Daka31211)
 - Register a new Application 
     Sign on URL is the root address of new app: https://localhost:44363/
     App Id URI: https://ColinCookOwinDemo.onmicrosoft.com/WebApplication1
 - View its details, make a client id

Your Microsoft account is the owner, but it cannot be used to log in with an application


CONFIGURING OWIN TO USE AZURE ACTIVE DIRECTORY AS A SOURCE
==========================================================

 - Make the Contact action protected by adding [Authorize] attribute, try going to it

 - Install-Package Microsoft.Owin.Security.Cookies
 - Install-Package Microsoft.Owin.Security.OpenIdConnect

 - Add following code to OWIN startup file
            app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);
            app.UseCookieAuthentication(new Microsoft.Owin.Security.Cookies.CookieAuthenticationOptions());

            app.UseOpenIdConnectAuthentication(
                new OpenIdConnectAuthenticationOptions
                {
                    ClientId = "42bae9e8-1d6e-42a1-9244-4f7d5a814b06", // replace with your ClientId
                    Authority = "https://login.microsoftonline.com/common/",
                    TokenValidationParameters = new TokenValidationParameters
                    {
                        ValidateIssuer = false          //We're using our own issuer validation
                    }
                }
             );

 - Navigate to Contact action again. Try logging in as the user you created earlier (Bob), if getting 407, add proxy help
YOU MUST LOG IN FROM THE CORRECT SSL ADDRESS!

  <system.net>
      <defaultProxy useDefaultCredentials="true">
        <proxy
	  proxyaddress="http://devproxy.campus.ccn.ac.uk:9090" 
          bypassonlocal="True" usesystemdefault="true"
        />
      </defaultProxy>
  </system.net>



CHECKING OUT THE CLAIMS AUTHENTICATION GIVES US
===============================================

        [Authorize]
        public JsonResult Claims()
        {
            var ci = new ClaimsPrincipal(System.Web.HttpContext.Current.User);
            var c = ci.Claims;

            var niceClaims = c.Select(cl => new { Type = cl.Type, Value = cl.Value });

            return Json(niceClaims, JsonRequestBehavior.AllowGet);
        }


AND FINALLY, THE EASY WAY TO DO ALL THIS THANKS TO SUSAN
========================================================

Create new project, right click project, go to Configure Azure AD Authentication
