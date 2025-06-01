# How news websites update history on back button

Source: [reddit](https://www.reddit.com/r/explainlikeimfive/comments/rgpduj/eli5_how_do_some_websites_hijack_my_back_button/)

Browsers provide JavaScript API to update history.

[Mozilla](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) - From The **`pushState()`** method of the [`History`](https://developer.mozilla.org/en-US/docs/Web/API/History) interface adds an entry to the browser's session history stack.

[Chrome](https://developer.chrome.com/docs/extensions/reference/api/history) - Use the `chrome.history` API to interact with the browser's record of visited pages. You can add, remove, and query for URLs in the browser's history.

**Pros**

* Useful on websites where user may want to stay within the website on back instead of leaving the website altogether.
  * If I'm reading my email and hit the back button, I usually just want to go back to my inbox instead of leaving the whole site ([copy-pasta from reddit](https://www.reddit.com/r/explainlikeimfive/comments/rgpduj/comment/holnt63)).

**Cons**

Annoyance on news websites where they override the back button navigation to go to their clickbait page instead of the actual page we came from.
