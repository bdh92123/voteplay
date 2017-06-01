# Voteplay

This is a simple firebase web application that users can vote.

Website: https://voteplay-5a588.firebaseapp.com

### Using

- Firebase 4
  - Authentication
  - Database
  - UI Paginator
- JQuery 1.11.2
- Bootstrap 3
  - Bootstrap Dialog
- Handlebars
  - x, z, xif, zif extension.



### Firebase Database

- Get votes

  ```javascript
  var ref = db.ref("votes");
  var votePaginator = new FirebasePaginator(ref, options);
  ..
  var changeValue = function(){
      // Render page of votes to view
      addPage(votePaginator.keys.reverse());
      // Need more button ?
      if(votePaginator.isLastPage || votePaginator.pageCount == 0) {
          votePaginator.off('value', changeValue);
          $("#more-btn").hide();
      } else {
          $("#more-btn").show();
      }
  };
  votePaginator.on('value', changeValue);
  ```

- Add vote

  ```javascript
  function addVote() {
    var voteObject = $("#frmVoteAdd").serializeObject();
    // To store created time of vote, ServerValue is used.
    voteObject["date"] = firebase.database.ServerValue.TIMESTAMP;
    voteObject["user"] = auth.currentUser.uid;
    voteObject["count"] = {
        "count1": 0,
        "count2": 0,
        "count3": 0,
        "count4": 0,
        "count5": 0,
        "user": {}
    }
    var key = db.ref().child('votes').push().key;

    // Write the new vote's data simultaneously in the vote list and the user's vote list.
    var updates = {};
    updates['/votes/' + key] = voteObject;
    updates['/user-votes/' + auth.currentUser.uid + '/' + key] = voteObject;

    return db.ref().update(updates, function(error) {
        if (error) {
            BootstrapDialog.show({
                type: BootstrapDialog.TYPE_DANGER,
                title: '오류',
                message: '알수없는 문제가 발생하였습니다.',
                buttons: [{
                    label: '닫기',
                    action: function(dialog){dialog.close();}
                }]
            });
        } else {
            $("#voteModal").modal("hide");
            document.location.reload();
        }
    });
  }
  ```

- Voting

  ```javascript
  function vote(btn, index) {
    ...

    var key = $(btn).data("key");
    console.log(key + ", " + index);
    var countRef = db.ref("/votes/" + key + "/count");
    var uid = auth.currentUser.uid;

    countRef.transaction(function(post) {
        if (!post)
            post = {};
        if (!post.user) {
            post.user = {};
        }

        if(post.user[uid] && post.user[uid] == index) {
            post["count" + post.user[uid]]--;
            post.user[uid] = null;
            return post;
        }

        if(!post["count" + index])
            post["count" + index] = 0;

        if (post.user[uid]) {
            post["count" + post.user[uid]]--;
        }

        post["count" + index]++;
        post.user[uid] = index;

        return post;
    }, function(){
        var value = db.ref("/votes/" + key);
        value.once('value', function(snapshot){
            $(btn).parents(".vote-item").replaceWith(
                $(renderVoteItem(key, snapshot.val())));
        });
    });
    return true;
  }
  ```

- Delete vote

  ```javascript
  function deleteVote(btn) {
    var key = $(btn).data("key");
    var updates = {};
    // Delete exists vote and user vote data.
    updates['/votes/' + key] = null;
    updates['/user-votes/' + auth.currentUser.uid + '/' + key] = null;
    // Used update function to set null for deleting data.
    db.ref().update(updates, function(error) {
        if (error) {
            BootstrapDialog.show({
                type: BootstrapDialog.TYPE_DANGER,
                title: '오류',
                message: '알수없는 문제가 발생하였습니다.',
                buttons: [{
                    label: '닫기',
                    action: function(dialog){dialog.close();}
                }]
            });
        } else {
            document.location.reload();
        }
    });
  }
  ```
  ​