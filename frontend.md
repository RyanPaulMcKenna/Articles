### Building a full-stack micro-service application in TypeScript for development & production 

------

## Introduction

------

This is the first article in a vast multi-part tutorial series in which we will build a full-stack micro-service application in TypeScript that is well designed for development and production. The aims of this project are to adhere; to SOLID design principles, adhere closely to the Twelve Factor App methodology.

------

### Part 1: The frontend

Prerequisites:

- NPM (Node Package Manager)

- Node.JS (JavaScript Runtime)

- A good text editor like VS Code

  ------

  Open up a terminal and run the following commands to create a react application that uses TypeScript. 

```
npx create-react-app frontend --template typescript
cd frontend
npm start // Verify that the app is working.
// Run CTR+C to stop the development server
npm eject
```

The create-react-app cli (Command-line-Interface) is actually giving us much more than just a basic react app with TypeScript enabled as we shall explore momentarily. Now run the command below to eject the project from create-react-app.

```
 run eject
```

Your new directory structure should look similar to this.

```
/config
	/jest
	env.js
	getHttpsConfig.js
	..several other files
/node_modules
/public
/scripts
	build.js
	start.js
	test.js
/src
	App.tsx
	..Various other React files
.gitignore
package-lock.json
package.json
README.md
tsconfig.json
```

Excellent now that that is done you can review the package.json and other files in the project and see just how much create-react-app was doing behind the scenes. Let take a moment to appreciate that.

###### Create-react-app:

- setups:

  - Webpack  (Bundler)

  - Babel (Tanspiler & Polyfiller)

  - Jest (Test Runner)

  - TypeScript

  - DotEnv (Manages environment variables for node)

  - Http and Https config (We just need to add the SSL certificate files and Https is enabled)

  - Provide node the correct shutdown signal to gracefully exit.

    - ```javascript
      ['SIGINT', 'SIGTERM'].forEach(function (sig) {
      	process.on(sig, function () {
          	devServer.close();
          	process.exit();
          });
      });
      ```

The configuration of Babel, Jest, TypeScript and basically everything else you can see in the project is all handled by Webpack. Webpack pre-processes all of these dependencies to produce an optimized production build. 

In fact when we add further dependencies to this project we will need to tell Webpack how to do their pre-processing via further plugins and transformers which we will referenced in Webpack.config.js.

------

## Frameworks and Design Patterns

It is important to note that React.js is a library, not a framework. Frameworks provide structure to the way developers add new features and help to reduce the amount of errors the software gains over time. Frameworks basically enforces a software design pattern for developers to follow. However we can implement well-known patterns ourselves to achieve a similar level of structure.

------



### Repository-Service Pattern

The repository-service pattern is a great pattern to provide our React  components with data. It allows us to move a lot of logic out of the react components and split it between several classes that are responsible for collecting a certain type of data. We can enforce the specification of this data with interfaces. All of this data can be accessed by the service class that we inject into the top-level React component. The service is composed of several of one or more repositories. All custom types/object should have their own interface. Here is a very simple example in UML.

 ![frontendUML](https://github.com/RyanPaulMcKenna/Articles/blob/970dea2d439607abd9f7be3e6b7049affb53b9b9/frontendUML.svg)

This example has three repository classes, three repository interfaces, four object classes, four object interfaces, one service class, one service interface.

Let implement something similar to this in our project using TypeScript. We're going to add an even simpler version of the system above for a UserList feature. Add the following folders and files so that your directory structure looks like as follows:

```
/config
/node_modules
/public
/scripts
/src
+/components
	+UserDetail.tsx
	+UserList.tsx
+/interfaces
	+interfaces.ts
+/repositories
	IUserRepository.ts
	UserRepository.ts
+/services
	+IUserService.ts
	+UserService.ts
.gitignore
package-lock.json
package.json
README.md
tsconfig.json
```

The "I's' preceding certain file names tell us that it the interface and will have a corresponding implementation class of the same name minus the "I". There tend to be many interfaces that describe the data that you are bringing into the application. These will be stored in  ./interfaces/interfaces.ts

### Interfaces

Add the following code to interfaces.ts

```typescript
interface ObservableUser {
    id: number;
    name: string;
    surname: string;
    createdAt: Date;
    updatedAt: Date;
}

interface IUser {
    id: number;
    name: string;
    surname: string;
    createdAt: string;
    updatedAt: string;
}

export { ObservableUser, IUser };
```

 Note : The "ObservableUser's" last two attributes are of type Date rather than type String as in the "IUser".

### Repositories

Add the following code to IUserRepository.ts and UserRepository.ts respectively.

```typescript
import { IUser } from "../interfaces/interfaces";

interface IUserRepository {
    getUsers(): Promise<IUser[]>;
}

export default IUserRepository;
```

```typescript
import { IUser } from "../interfaces/interfaces";
import IUserRepository from "./IUserRepository";

class UserRepository implements IUserRepository {
    public URL_ENDPOINT: string = '/api/users';
    public async getUsers(): Promise<IUser[]> {
        try {
            const response = await fetch(this.URL_ENDPOINT, {
                method: 'GET',
                headers: {
                    'Accept': 'application/json',
                    'Content-Type': 'application/json'
                }
            });
            if (response.status >= 200 && response.status < 300) {
                const jsonResponse = response.json();
                return jsonResponse;
            } else {
                throw new Error(response.statusText);
            }
        } catch (error) {
            throw error;
        }
    }
}

export default UserRepository;
```

The API that this calls does not exist yet, we will be creating the backend API in later part of this series. If you want the satisfaction of seeing the code display at the end then TEMPORARILY swap the code in UserRepository with this mocked version. 

```typescript
import { IUser } from "../interfaces/interfaces";
import IUserRepository from "./IUserRepository";

class UserRepository implements IUserRepository {
    public URL_ENDPOINT: string = '/api/users';
    public async getUsers(): Promise<IUser[]> {
	let AllUsers: IUser[]  = [
        {
            id: 1,
            name: 'Darth',
            surname: 'Vader',
            createAt: "'01 Jan 1970 00:00:00 GMT'", //unix time stamp
            UpdatedAt: "'01 Jan 1970 00:00:00 GMT'", //unix time stamp
        },
        {
            id: 2,
            name: 'Luke',
            surname: 'Skywalker',
            createAt: "'01 Jan 1970 00:00:00 GMT'", //unix time stamp
            UpdatedAt: "'01 Jan 1970 00:00:00 GMT'", //unix time stamp
        }
    ];
        return Promise.resolve(AllUsers);
    }
}

export default UserRepository;
```



### Services

Add the following code to IUserService.ts and UserService.ts respectively.

```typescript
import { ObservableUser } from "../interfaces/interfaces";

interface IUserService {
    getUsers(): Promise<ObservableUser[]>;
}

export default IUserService;
```

```typescript
import { IUser, ObservableUser } from "../interfaces/interfaces";
import IUserRepository from "../repositories/IUserRepository";
import IUserService from "./IUserService";

class UserService implements IUserService {
    constructor(private userRepository: IUserRepository) {

    }
    private observableUser(user: IUser): ObservableUser {
        const makeDate = (date: string): Date => new Date(date);
        return {
            id: user.id,
            name: user.name,
            surname: user.surname,
            createdAt: makeDate(user.createdAt),
            updatedAt: makeDate(user.updatedAt),
        };
    }
    
    public async getUsers(): Promise<ObservableUser[]> {
        try {
            const users = await this.userRepository.getUsers();
            return users.map(user => this.observableUser(user));
        } catch (error: any) { 
            throw new Error(error);
        }
    }
}

export { UserService };

export default UserService;
```

### Components

Add the following code to UserDetail.tsx, UserList.tsx, App.tsx (./src/App.tsx) respectively.

```react
import React from "react";
import { IUser } from "../interfaces/interfaces";

interface IProps {
    user: IUser;
}

export const UserDetail = (props: IProps) => (
    <div>
        <h4>User: {props.user.id}</h4>
        <ul>
            {Object.keys(props.user).map((key: string) => <li> {key}: {props.user[key]} </li>)}
        </ul>
    </div>
);
```

```react
import React from "react";
import { IUser } from "../interfaces/interfaces";
import IUserService from "../services/IUserService";
import { UserDetail } from "./UserDetail";

interface IProps {
    service: IUserService;
}

interface IState {
    users: IUser[];
}

export default class UserList extends React.component<IProps, IState> {

    public async componentDidMount(){
        const users = await this.props.service.getUsers();
        this.setState({ users });
    }

    public render() {
        const userList: IUser[] = this.state.users;
        return (
            <div>
                {userList.map((user)=> <UserItem user={user} />)}
            </div>
        );
    }
}
```

```react
import React from "react";
import "./App.css";


import { UserService } from "../services/User.service";
import { IUserService } from "../services/IUser.Service";
import { UserRepository } from "../repositories/User.repository"

const service: IUserService = new UserService(new UserRepository());


class App extends React.Component{

  public render() {
    return (<div>
      <UserList service={service} />
    </div>);
  }
} 


export default App;
```

Excellent, we have implemented a simple Repository-Service pattern! However there are often situations where we will need even more fine grained control over how data is brought into the application. For example we could add an extra folder at the top-level of our folder structure for DAO's (Data Access Objects). These DAO's would be the first code to interact with our data after the repository has collected it from the server. The DAO should take the data as a parameter and output a new object that contains the bare minimum that the feature requires of it. This solution is an answer to a common problem in development in which the endpoint being called provides more data than is needed. The DAO allows you to throw away the extra data as soon as possible, so your not lugging it around through your entire application. If your calling endpoints that bring in more data than required. Then this design choice will help mitigate the damage to performance the application is suffering from. Ideally the backend API should provide the exact data the client required or data that is not much larger.

------



### Conclusion

In conclusion we have made a great start to our frontend application. In the next part of this series we will be  implementing Behaviour-driven development, a superset of Test-driven-development that will give us a well-defined workflow for creating new features in a test-driven way according to the goals, use-cases, stories and requirements of the new feature being implemented. In addition we will be adding Storybook.js, a excellent tool prototyping the appearance of UI components without needing to put them in the application before your confident the component is ready. 

That's all for now, thanks for reading and I'll see you next time!
