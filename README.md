
# <img src="https://avatars0.githubusercontent.com/u/13003405?v=3&s=100" height="50" style="position: relative; top: -5px;" alt=""> uniJS

uniJS is a library for rendering [ReactJS](https://github.com/facebook/react) apps on [node.js](https://github.com/joyent/node).

* **Universal:** Use the same code base for server- and client-rendering. Also there is no force to use flux.
* **Autofetch Data:** Don't worry about data fetching on your server. Just provide a REST-API somewhere and perform AJAX-Requests. uniJS automatically detects which data is neccessary for rendering and fetches it before render.
* **State Sync:** Sync's the server rendered state to the client. After rendering the app uniJS takes the state out of all components and responds them with the HTML to the client. There the state gets pushed back to the components as initial state.

<br>
> *uniJS requires to use [react-router](https://github.com/rackt/react-router).*


##Demo
* **Live:** [uniJS on Heroku](https://unijs.herokuapp.com/)<br>
 (it's free account so it may takes some time when the app sleeps)
* **Repo:** [unijs/unijs-demo](https://github.com/unijs/unijs-demo)


##Usage

###Installation:

`npm install unijs`

###Server:

On server-side you need to define your react-router Routes by setting the Router. Also you need to specify the URL of your REST-API Server.

```js
var Router = require('client/js/Routes.js')
var unijs = require('unijs');
var Server = unijs.Server();

var unijsApp = new Server.App('myDemoApp');

unijsApp.Router = Router;
unijsApp.resources.push(__dirname+'/bundle.js');
unijsApp.setApiUrl('http://localhost:5000/');

Server.mount('/', unijsApp);
```

####uniJS-builder
uniJS-builder simplifies the usage of uniJS. By defining the path of your Routes file it compiles with babel, browserify and uglify. Then it adds the bundle to the resources.

```js
var unijs = require('unijs');
var Server = unijs.Server();
var App = unijsBuilder.extend(Server.App);

var unijsApp = new App('myDemoApp');

unijsApp.routesPath = 'client/js/Routes.js';
unijsApp.setApiUrl('http://localhost:5000/');

Server.mount('/', unijsApp);
```

###Client:

On client you need to extend each component that has state or is loading data with AJAX.

```js
class Blog extends React.Component{
	// component methods
}

Blog = unijs.extend(Blog);
```

And load your data with superagent plus unijs-plugin in the componentDidMount method. (componentDidMount gets called while server-rendering when componentWillMount gets called)

```js
class Blog extends React.Component{
	componentDidMount(){
		superagent.get('/getdata')
			.use(unijs.superagentPlugin)
			.end(function(err, res) {
				this.setState(JSON.parse(res.text));
			}.bind(this)
		);
	}
}
```

Define your initial state as you do it in React ES6:

```js
class Blog extends React.Component{
	componentDidMount(){
		superagent.get('/getdata')
			.use(unijs.superagentPlugin)
			.end(function(err, res) {
				this.setState(JSON.parse(res.text));
			}.bind(this)
		);
	}
}
```

A basic component that loads data and holds state looks like that:

```js
var React = require('react');
var superagent = require('superagent');
var unijs = require('unijs');

class Blog extends React.Component {
	constructor() {
		super();
		this.state = {title: '', content: ''};
	}
	componentDidMount() {
		superagent.get('/blog/getpost/' + this.props.id)
			.use(unijs.superagentPlugin)
			.end(function(err, res) {
				this.setState(JSON.parse(res.text));
			}.bind(this)
		);
	}
	componentWillReceiveProps(nextProps) {
		if (nextProps.id != this.props.id) {
			this.loadBlogPost(nextProps.id);
		}
	}
	render() {
		return (
			<div>
				<h1>{this.state.title}</h1>
				<div>{this.state.content}</div>
			</div>
		);
	}
}
Blog = unijs.extend(Blog);

module.exports = Blog;
```

The `Routes.js` file defines all routes of your app with react-router.

```js
var React = require('react');
var Route = require('react-router').Route;

var BlogPost = require('./components/BlogPost.react');

var routes = (
	<Route name="main" path="/">
		<Route path="/blogpost/:id" name="blogpost" handler={BlogPost}/>
	</Route>
);

module.exports = routes;
```

###Contribution and Ideas are Welcome! ;-)
Ideally just create a GitHub Issue to discuss or create a PR.

##What changed compared to a default ReactJS app?
- Use the `uniJS.loadMixin` mixin in all data loading components
- Use the `uniJS.stateMixin` mixin in all components using state
- Use [superagent](https://github.com/visionmedia/superagent) to load your data
- Use the `uniJS.superagentPlugin` plugin to enable universal rendering of this request

##Performance Note:
To be able to render your app as fast as possible try to map your route params to your api calls. Do not convert them in any way.

###Do:
`:id` and `:time` are placeholders for some kind of id and a timestamp

React-Router URL | API Calls
 --- | ---
/blog/:id | /loadpost/:id
/blog?id=:id | /loadpost/:id
/blog/:id | /loadpost?id=:id
/blog/:id?time=:time | /loadpost?id=id&time=:time

###Do NOT !!!!:
React-Router URL | API Calls
 --- | ---
/blog/:id/:time | /loadpost/:id/[:time/2]

`[:time/2]` intents to describe the calculated half of `:time`



[Apache License Version 2.0](LICENSE)
