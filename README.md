# modstr-snippets
Snippets of code that I wrote for Modstr, a mobile social networking app for car enthusiasts.

First, this is an example that loads the current user's social posts, appends them
to a div so the user can see them, and enables infinite scrolling. Infinite
scrolling means that only a few (5 in this case) posts are loaded each time the
user scrolls to the bottom of the page.
```javascript
let myPostContainer = $$('#my-posts-container');

const postsToLoadPerInfinite = 5;

let isInfiniteLoading = false;
let lastLoadedPostIndex = 0;
let userId = storage.get('user').id;

socialPost.createUserSocialPosts(userId, postsToLoadPerInfinite, lastLoadedPostIndex).then( posts => {
  myPostContainer.append(posts);
  Modstr.initImagesLazyLoad('#view-profile');
  lastLoadedPostIndex += postsToLoadPerInfinite;
});

$$('#view-profile .infinite-scroll').on('infinite', () => {

  //if already loading more posts, don't load more.
  if (isInfiniteLoading) return;

  isInfiniteLoading = true;

  socialPost.createUserSocialPosts(userId, postsToLoadPerInfinite, lastLoadedPostIndex).then( posts => {

    if(!posts) {
      //if out of posts, stop infinite scrolling and remove preloader.
      Modstr.detachInfiniteScroll($$('#view-profile .infinite-scroll'));
      $$("#view-profile .infinite-scroll-preloader").html("");
    } else {
      myPostContainer.append(posts);
      Modstr.initImagesLazyLoad('#view-profile');
      isInfiniteLoading = false;
      lastLoadedPostIndex += postsToLoadPerInfinite;
    }
  });
});
```

This snippet comes from the socialPost UI component module. It uses a
model to request posts from our server. Framework7's templating
engine is also used here to place the retrieved posts' data into
valid HTML.
```javascript
/*
Creates HTML for the user's social posts. Creates the numberOfRowsToGet most
recent posts, starting with the numberOfRowsToSkip'th row.
*/
function createUserSocialPosts(userId, numberOfRowsToGet, numberOfRowsToSkip) {
  return new Promise((resolve, reject) => {
    socialPostModel.getXPostsAfterIndexYByUser(userId, numberOfRowsToGet, numberOfRowsToSkip).then((data) => {
      if(data.length === 0) {
        resolve("");
        return;
      }

      resolve(compileSocialPostTemplate(data));
    });
  });
}
```

Here is an example MySQL query that our server uses to update data
in our database based on API calls made by the client app.
This API is what the client app uses to access our database.
```javascript
db.query(`UPDATE users SET imageURL = ?, userName = ?, firstName = ?, lastName = ?,
      updatedAt = NOW()
      WHERE id = ?`,
      [data.imageURL, data.userName, data.firstName, data.lastName, data.userId],
      (error, results, fields) => {

  const id = data.userId;
  db.query('SELECT * from users WHERE id = ?', [id], callback);
});
```

The following code utilizes Framework7's built in pull-to-refresh
functionality. This allows users to pull down near the top of the
screen in order to refresh the posts that they can see on their
home tab.
```javascript
$$('#home .pull-to-refresh-content').on("ptr:refresh", () => {
  setTimeout(() => {
    socialPostContainer.html("");

    socialPost.createSomeSocialPosts(lastLoadedPostIndex, 0).then( posts => {
      socialPostContainer.append(posts);
      Modstr.initImagesLazyLoad('#home');
    });

    Modstr.pullToRefreshDone();
  }, 1000);
});
```
Finally, here is an example of our HTML. The following is our sign
in page. We used Auth0 for user authentication, because it allowed
us to quickly implement a secure sign-in process that included
password reset functionality.
```html
<div class="login-screen">
  <div class="view">
    <div class="page">
      <div class="page-content login-screen-content">
        <div class="login-screen-title large-title modstr-font"><img src="../../../img/modstrLogo.png" alt="MODSTR Logo" /></div>
        <div class="tagLine">BUILD &nbsp; • &nbsp; SHARE &nbsp; • &nbsp; DRIVE &nbsp; • &nbsp; REPEAT</div>
        <form id="login-form">
          <div class="list-block">
            <ul class="light-transparent-bg">
              <li id="email" class="item-content sign-in create-account reset-password">
                <div class="item-inner">
                  <div class="item-input">
                    <input type="email" name="email" placeholder="Email">
                  </div>
                </div>
              </li>
              <li id="password" class="item-content password sign-in create-account">
                <div class="item-inner">
                  <div class="item-input">
                    <input type="password" name="password" placeholder="Password">
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <div class="feedback-messages">
            <p class="error"></p>
            <p class="success"></p>
          </div>
          <div class="btn-block">
            <button id="sign-in-btn" class="button button-fill button-big action-btn sign-in">
              Sign In
            </button>
            <button id="create-account-btn" class="button button-fill button-big action-btn create-account hidden">
              Create Account
            </button>
            <button id="reset-password-btn" class="button button-fill button-big action-btn reset-password hidden">
              Reset Password
            </button>
          </div>
          <div class="list-block">
            <div class="list-block-label">
              <p><a href="#" class="action-btn show-sign-up sign-in">Don't have an account? Sign up</a></p>
              <p><a href="#" class="action-btn show-sign-in hidden create-account reset-password">Have an account? Log in</a></p>
              <p><a href="#" class="action-btn show-reset-password sign-in" id="forgot-pass">Forgot password?</a></p>
            </div>
          </div>
        </form>
      </div>
    </div>
  </div>
</div>
```
