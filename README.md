# modstr-snippets
Snippets of code from Modstr, a mobile social networking app for car enthusiasts.

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
