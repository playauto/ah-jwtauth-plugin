# ah-jwtauth-plugin

Uses auth0 node-jsonwebtoken to allow token authentication of actions, It is a really slimmed down version of panjiesw's plugin over at https://github.com/panjiesw/ah-auth-plugin as I didn't want to make the assumptions around users and email systems. 
The original was developed by https://github.com/lookaflyingdonkey/ah-jwtauth-plugin and forked because of missing maintenance.

## Installation
* `npm install ifavo/ah-jwtauth-plugin --save`
* Add the "ah-jwtauth-plugin" plugin to your ActionHero `config/plugins.js`:

    exports['default'] = {
      general: function(api)
      {
        return {
          plugins: [
            'ah-jwtauth-plugin'
          ]
        };
      }
    };

## Usage
This plugin will check your action templates for a property called `authenticate`, if it exists and is true it will then require that a "Authorization" header has been sent with the request holding a valid JSON Web Token. I make use of the node-jsonwebtoken module (https://github.com/auth0/node-jsonwebtoken) to generate and validate the tokens.  
The value of the header must start with `Token ` to be picked up.

An example header would be:

    Authorization: Token eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpZCI6MTIzNCwiZW1haWwiOiJ0ZXN0QGV4YW1wbGUuY29tIiwiaWF0IjoxNDQ1Mjc2NDYyfQ.vB5yV2PGOj1oVIsqDDU7uWlkrf--a1TX2EtsqSDkjqyCGzho1rYO-AQgsDxsKcf5rocmeAx_4Jq4A3wHff2euQ

Test it with curl:

    curl http://localhost:8080/yourAction --header "Authorization: Token eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpZCI6MTIzNCwiZW1haWwiOiJ0ZXN0QGV4YW1wbGUuY29tIiwiaWF0IjoxNDQ1Mjc2NDYyfQ.vB5yV2PGOj1oVIsqDDU7uWlkrf--a1TX2EtsqSDkjqyCGzho1rYO-AQgsDxsKcf5rocmeAx_4Jq4A3wHff2euQ"

Token data sent via `Authorization` Headers will be provided in your action within `data._jwtTokenData`:

    […]
    run: function(api, data, next){
        data.response.tokenData = data._jwtTokenData;
        […]

### Settings
* You can select the algorithm you want to use with the options available at https://github.com/auth0/node-jsonwebtoken
* You will need to set a secret that will be used to create the token

### Generate a token
You need to generate a token once a user has successfully authenticated against your API, this could be by signing in with a username/password or you could simply generate them through a CMS, print them out and post them to your users ;-)
 
    api.jwtauth.generateToken({id: 1234, email: 'test@example.com'}, function(token) {

      // token will hold the generated token
      data.response.token = token;
      next();

    }, function(err) {

      // An error occured generating a token
      data.error = err;
      next();

    });
    
I would suggest not storing a huge amount of information in them as it will just mean more data transferred per request, but you can put some identifying info like email, name, etc. The beauty of this is that you don't need to hit the database every time to authenticate a user.
        
### Validate a token
While the plugin will automatically validate a token and put it on the connection object for you as connection.user, you can also validate the tokens yourself like below.

    api.jwtauth.processToken("abce1234==", function(data) {

      // Valid data, lets set it and continue
      console.log(data);

    }, function(err) {

      console.log('Error', err);

    });

## Todo
* Need to add in expiry, I personally dont use it but I may add it to my app with refreshing tokens in the future.
