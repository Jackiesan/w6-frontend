# Connecting the Frontend and the Backend

By the end of this lesson. You should be able to set up two separate servers that will speak with each other -- one with frontend code and the other with react code.

## Core Learning Objective

* Communicate with an application server using a front-end client

## Sub-Objectives

* Login a user via an external API and store a token in LocalStorage
* Logout a user locally
* Signup a user via an external API and store a token in LocalStorage
* Authorize routes on the frontend
* Populate information from an external API
* Delete records through an external API
* Create new records through an external API
* Update existing records through an external API

## Installation

1. Fork & clone this repository

1. `npm install`

1. `npm start`

## Instructions & Guiding Questions

- [ ] Start both your frontend server and your backend server. Then try copying the code below into the web console.
  ```js
  fetch('http://localhost:5000/api/users')
    .then(res => res.json())
    .then(console.log)
  ```

* **Question:** What error do you get? Why?

* **Your Answer:** I get a 401 unauthorized error and a CORS error because we are not whitelisted.

---

- [ ] To get around this issue, we need to explicitly allow for requests to come from `localhost:3000`. To do so, we will use the [cors](https://www.npmjs.com/package/cors) package. Install `cors` on the _backend server_ and whitelist `localhost:5000`.

* **Question:** Try your request again. What error do you get? Why?

* **Your Answer:** We get a 401 error that we are not authorized. We are getting that from the isLoggedIn function returning a 401 because we are not logged in.

---

- [ ] In `App.js`, we have created our `loginUser()` method. Try invoking that function through the frontend, inspecting what is outputted.

---

- [ ] We now want to try and login the user when they hit submit. Add the following to your `loginUser()` method:
  ```js
  fetch('http://localhost:5000/api/login', {
    body: JSON.stringify(user),
    headers: {
      'Content-Type': 'application/json'
    },
    method: 'POST',
  }).then(res => res.json()).then(console.log)
  ```

* **Question:** Why do we need to include the "Content-Type" in the headers?

* **Your Answer:** We need to include the content type in the headers so that our server knows that the information it's getting is json. Server is expecting json.

* **Question:** How could you convert this method to an `async` method? We can add it before the function name loginUser.

---

- [ ] Let's move our requests to a better place. Create a new file at `./src/api/auth.js`. Add the following inside of it:
  ```js
  const { NODE_ENV } = process.env
  const BASE_URL = NODE_ENV === 'development'
    ? 'http://localhost:5000'
    : 'tbd' // Once we deploy, we need to change this

  export const login = async (user) => {
    const response = await fetch(`${BASE_URL}/api/login`, {
      body: JSON.stringify(user),
      headers: {
        'Content-Type': 'application/json'
      },
      method: 'POST'
    })
    const json = await response.json()

    return json
  }
  ```

  Update `App.js` to use the `login()` function, logging out the response from it.

* **Question:** What is happening on the first couple of lines of the new file you've created?

* **Your Answer:** The base url of development is being set at 'http://localhost:5000'. If it's not development then it's 'tbd'.

---

- [ ] Let's store the token in LocalStorage with a key of `journal-app`.

* **Question:** Why are we storing the token?

* **Your Answer:** We are storing the token because now the token will be saved across browser sessions.

---

- [ ] We now have the token, but we don't have any of the user information. Add a new function to our `./src/api/auth.js` called `profile()` that sends over the token in order to retrieve the user information. Then, log that information.

* **Question:** Where did you write your code to manipulate LocalStorage? Why?

* **Your Answer:** I wrote it in the login and signup methods so that the token can be saved as soon as the user logs in.

---

- [ ] Now that we have the user's information, let's store the user's ID in state. Set `currentUserId` to the user ID you've retrieved.

* **Question:** What changes on the page after you successfully login? Why?

* **Your Answer:** The Navigation bar changes once successfully logged in because we defined in the component that if there is a currentUser we render the authenticated links.

* **Question:** What happens if you enter in the incorrect information? What _should_ happen?

* **Your Answer:** The request should return with a status of a 401 and show an error.

---

- [ ] Try refreshing the page. You'll notice it _looks_ like you've been logged out, although your token is still stored in LocalStorage. To solve this, we will need to plug in to the component life cycle with `componentDidMount()`. Try adding the following code to `App.js`:
  ```js
  async componentDidMount () {
    const token = window.localStorage.getItem('journal-app')
    if (token) {
      const profile = await auth.profile()
      this.setState({ currentUserId: profile.user._id })
    }
  }
  ```

* **Question:** Describe what is happening in the code above.

* **Your Answer:** When the component mounts for the first time, the app searches for the token stored in localStorage and if there is one then the currentUserId is saved in the component's state.

---

- [ ] Now when you refresh the page, it looks as though you are logged in. Next, try clicking the logout button.

* **Question:** When you click "Logout", nothing happens unless you refresh the page. Why not?

* **Your Answer:** Nothing happens because the app still has the current user stored in it's state.

---

- [ ] Update the `logout()` method to appropriately logout the user.

* **Question:** What did you have to do to get the `logout()` function to work? Why?

* **Your Answer:** You have to remove the item journal-app from the local storage and reset the state of the variable currentUserId.

---

- [ ] Following the patterns we used above, build the Signup feature.

---

- [ ] When a user logs in or signs up, we should bring them to the `/users` route. Update both features so that the user is moved to that route after a successful login/signup.

---

- [ ] Try logging out and then go directly to the `/users` route.

* **Question:** What happens? What _should_ happen?

* **Answer:** What should happen is that since the user is not authenticated they should not be able to see the page. We updated the code to redirect them to the login page if they are not authenticated and try to access this route.

---

- [ ] Try _replacing_ the `/users` Route in `App.js` with the following:
  ```jsx
  <Route path='/users' render={() => {
    return this.state.currentUserId ? <UsersContainer /> : <Redirect to='/login' />
  }} />
  ```

* **Question:** Describe what is happening in the code above.

* **Your Answer:** If the state of the app has a current user then the UsersContainer component will render else the router will redirect to the login page.

---

- [ ] Now try logging in. Then, when you're on the `/users` page, refresh the page.

* **Question:** What happens and why?

* **Your Answer:** I can't quite remember what was happening before we changed the code.

---

- [ ] To solve this problem, let's add a `loading` key to our App's state, with the default value set to `true`. When `componentDidMount()` finishes, set the `loading` key to equal `false`. Using this key, solve the issue of refreshing on the `/users` page. Make sure everyting continues to work whether you are logged in or out.

* **Question:** What did you do to solve this problem?

* **Your Answer:** We added a variable to our state that is a boolean indicating whether the app is loading.

---

- [ ] We will have the same problem on the `/users/<userId>/posts` page. Use the same strategy to have this page load correctly on refresh.

* **Question:** In what component did you add the `loading` property and why?

* **Your Answer:** I added it to the container for list of posts because it's the component responsible for rendering a list of posts.

---

- [x] Using the same principals as above, make it so that if the user is logged in, they cannot go to the `/login` or `/signup` routes. Instead, forward them to `/users`.

---

- [x] Right now, the data inside of `users/Container.js` is static. Using `componentDidMount()`, update this code so that we pull our data from our API.

  _NOTE: You may want to create a new file in `./src/api/` to organize these requests.

---

- [x] Let's get our "Delete" link working. On the backend, create a `DELETE Post` route with the path of:
  ```
  DELETE /users/:userId/posts/:postId
  ```
  This request should only be able to be made if the user is logged in and it's the user's post.

---

- [x] On the frontend, create a new function in your `src/api` folder that will delete a post. Use that function inside of the `src/components/posts/Container` file. Upon successful deletion, send the user back to the `/users/<userId>/posts` route.

---

- [x] Try deleting a post using the link.

* **Question:** Why did the number of posts not change when you were redirected back to the `/users` route?

* **Your Answer:** Whenever we modify our data with a Create, Update, or Delete, we have a few options on how to make our frontend reflect those changes. What options can you think of?

* **Question:** The number of posts did not change because the app did not make another request to the server. There are two options that I can think of. The first is to have the list of posts container to make another service request. Another is to have the destroyPost change the state of the posts container to reflect that one post has been deleted.

---

- [x] Using your preferred method, update your code so that the frontend will reflect the changes made to the backend.

---

- [ ] Right now it looks like we can Edit and Delete posts for other users. Hide/display those actions to only be available on a post if it's the user's post.

---

- [ ] Let's get our "Create a New Post" form to work. On the backend, create a `CREATE Post` route with the path of:
  ```
  POST /users/:userId/posts
  ```
  This request should only be able to be made if the user is logged in and it's from the same user as the one specified in the route.

---

- [ ] On the frontend, build a function that will POST to the database. Connect that function to the `onSubmit` functionality for the creation form. Finally, use your preferred method to update the state of our frontend. Upon successful creation, send the user back to the `/users/<user-id>/posts` page.

---

- [ ] Our final step is to get our Update form to work. Follow the steps from above to finish this final feature.

## Exercise

We got a lot done but there's still a lot to do to make this app fully functional. Complete the following features on this application.

- [ ] If there are no posts for a user, show a message on their `/users/<userId>/posts` page that encourages them to create a new post.

- [ ] If there is no emotion for a post, hide the associated message on each post.

- [ ] Show the user's username on the navigation when they are logged in as a link. When clicked, go to a new page: `/users/<userId>/edit`

- [ ] Create a page at `/users/<userId>/edit` that allows a user to update their `name`. On save, redirect them to their `/users/<userId>/posts` page.

- [ ] If the user has a name, show that on the Navigation, `/users` page, and `/users/<userId>/posts` page instead.

- [ ] On the login page, appropriately handle errors so that the user has a chance to correct their username/password combination. Display some kind of helpful message.

- [ ] On the signup page, appropriately handle errors so that the user has a chance to correct their username/password combination. Display some kind of helpful message.

- [ ] On the create post page, appropriately handle errors so that the user has a chance to correct their post. Display some kind of helpful message.

- [ ] On the update post page, appropriately handle errors so that the user has a chance to correct their post. Display some kind of helpful message.

- [ ] Create a new frontend route at `/users/<userId>/posts/<postId>` that shows a single post. Update your Create and Edit forms to redirect here instead of to the general `/posts` page.

