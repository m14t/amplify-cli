
## Amplify CLI React Tutorial
This tutorial will walk you through using the AWS Amplify CLI with a React application. A similar process could be used with a React Native application (you would omit hosting) or alternative JavaScript frameworks.


# Installation
- An AWS Account and credentials are necessary. See [LINK]().
- Ensure you have NodeJS installed
  - Examples below use `yarn` but `npm` works as well.
- Download [LINK](https://amplify-cli-beta-3dd89264-8c7c-11e8-9eb6-529269fb1459.s3.amazonaws.com/amplify-cli-0.1.0.tgz?AWSAccessKeyId=AKIAIFBGMKZSDB7FRP7A&Signature=l8ieJ9wqXDJN8TJ02D63Taxmgjo%3D&Expires=1533366622)
- Navigate into the unpacked directory and run `npm run setup-dev`
- Ensure you have [Create React App](https://github.com/facebook/create-react-app) installed and create a new project:
```
yarn create-react-app -g
create-react-app myapp
cd myapp
```

# Familiarizing yourself with the CLI
To get started, inside the new directory initialize your project:
`amplify init`

After answering a few questions

`amplify help` can be used at any time to see the overall command structure, and `amplify help <category>` to see actions for a specific category. 

The Amplify CLI uses Amazon CloudFormation under the covers, and you can add/modify configurations locally before pushing them for execution in your account. To see the status of the deployment at any time type `amplify status`.

# Publish your web app

Without making any changes to your React application add in web hosting:
`amplify add hosting`.

You'll notice some questions are asked such as the bucket name or application files. You can use the defaults for most of these by pressing **Enter**.

**Note**: Adding or removing category features has an order alias for convenience. Typing `amplify hosting add` would also work.

You can see the changes are not deployed by running `amplify status` and then simultaneously build & deploy your site with `amplify publish`. Once complete your application will be visible from an S3 hosting bucket for testing, and it is also fronted with a CloudFront Distribution.

# Add User Registration and Sign-In

Now that your app is in the cloud, you can add some features like allowing users to register for your site and login. Run `amplify add auth` and select the **Default**

Next, add the Amplify library to your application:
```
yarn add aws-amplify aws-amplify-react
```

You should see a `/src/aws-exports.js` file created which will have all of the appropriate cloud resources defined for your application. Edit `./src/App.js` to include the Amplify library, configs, and [React HOC](https://reactjs.org/docs/higher-order-components.html). Then initialize the library:

```
import Amplify from 'aws-amplify';
import aws_exports from './aws-exports';
import { withAuthenticator } from 'aws-amplify-react';
Amplify.configure(aws_exports);
```

Then wrap the default `App` component using `withAuthenticator` at the bottom of the file:

```
export default withAuthenticator(App, true);
```

You can now use `amplify publish` to build and publish your app again. This time you'll be able to register a new user and sign-in before entering the main application.

# Add Analytics and Storage

A login screen is nice but now is the time to add some features, like tracking analytics of user behavior and uploading/downloading images in the cloud. Start by running `amplify add analyitcs` in your project. Then run `amplify add storage` and select the S3 provider. When complete run `amplify push` and the cloud resources will be created.

Edit your `App.js` file in the React project again and modify your imports so that the `Analytics` and `Storage` categories are included as well as the `S3Album` component, which will be used for uploading and downloading photos.

```
import { Analytics, Storage } from 'aws-amplify';
import { withAuthenticator, S3Album } from 'aws-amplify-react';
```

The `Analytics` category will automatically track user session data such as sign-in events, however you can record custom events or metrics at any time. You can also use the `Storage` category to upload files to a private user location after someone has logged in. First, add the following line after `Amplify.configure()` has been called:

`Storage.configure({ level: 'private' });`

Next, add the following methods before the component's `render` method:

```
  uploadFile = (evt) => {
    const file = evt.target.files[0];
    const name = file.name;

    Storage.put(name, file).then(() => {
      this.setState({ file: name });
    })
  }

 componentDidMount() {
    Analytics.record('Amplify_CLI');
  }
```

Finally, modify the `render` method so that you can upload files and also view any of the "private" photos that have been added for the logged in user:

```
  render() {
    return (
      <div className="App">
        <p> Pick a file</p>
        <input type="file" onChange={this.uploadFile} />
        <S3Album level="private" path='' />
      </div>
    );
  }
```
Refresh the web page manually to see the uploaded image in your app.

Save your changes and run `amplify publish`. Since you already pushed the changes earlier just the local build will be created and uploaded to the hosting bucket. Login as before if necessary and you'll be able to upload photos, which are protected by user. You can also refresh the page to view them.


# Add REST API calls to a database

Now that your application is setup, the final piece is to add a backend API with data that can be peristed in a database. For this example we will use a REST backend with a NoSQL database. Run `amplify add api` and follow the prompts, giving your API a friendly name such as **myapi** or something else that you remember. Use the default `/items` path and select **Create a new lambda function**. Select the option titled **CRUD function for Amazon DynamoDB table (Integration with Amazon API Gateway and Amazon DynamoDB)** when prompted. This will create an architecture of Amazon API Gateway with Express running in a Lambda function that reads and writes to Amazon DynamoDB. You'll be able to later modify the routes in the Lambda function to meet your needs and update it in the cloud. 

Since you do not have a database provisioned yet, the CLI workflow will prompt you for this. Alternatively, you could have run `amplify add storage` beforehand to create a DynamoDB table and use it in this setup. When the CLI asks you for the Primary Key structure use an attribute named `id` of type `String`.
Next, you would need to select the security type for the API. Select **Authenticated - AWS IAM (Signature Version 4 signing** option.


Edit your `App.js` file in the React project again and modify your imports so that the `API` category is included as well to make API calls from the app.

```
import { Analytics, Storage, API } from 'aws-amplify';
```


In `App.js` add in the following code before the `render()` method, update **myapi** if you used an alternative name during setup:
```
  post = async () => {
    console.log('calling api');
    const response = await API.post('myapi', '/items', {
      body: {
        id: '1',
        name: 'hello amplify!'
      }
    });
    alert(JSON.stringify(response, null, 2));
  }
  get = async () => {
    console.log('calling api');
    const response = await API.get('myapi', '/items/object/1');
    alert(JSON.stringify(response, null, 2));
  }
  list = async () => {
    console.log('calling api');
    const response = await API.get('myapi', '/items/1');
    alert(JSON.stringify(response, null, 2));
  }
  ```

Update the `render()` method to include calls to these three methods:
```
  render() {
    return (
      <div className="App">
        <p> Pick a file</p>
        <input type="file" onChange={this.uploadFile} />
        <button onClick={this.post}>POST</button>
        <button onClick={this.get}>GET</button>
        <button onClick={this.list}>LIST</button>
        
        <S3Album level="private" path='' />
      </div>
    );
  }
  ```

Save the file and run `amplify publish`. After the API is deployed along with the Lambda function and database table, your app will be built and updated in the cloud. You can then add a record to the database by clicking **POST** and use **GET** or **LIST** to retrieve the record, which has been hard coded in this simple example.

In your project directory, open `./amplify/backend/function` and you will see the Lambda function that you created. The `app.js` file runs the Express function and all of the HTTP method routes are available for you to manipulate. For instance the `API.post()` in your React app corresponded to the `app.post(path, function(req, res){...})` code in this Lambda function. If you choose to customize the Lambda you can always update it in the cloud with `amplify push`. 