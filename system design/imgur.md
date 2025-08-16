## users table

* username
* first_name
* last_name
* email
* password
* location
* phone
* age
* created_on
* updated_on
* is_influencer

Partitioned by `user_id`.

## posts table

* post_id
* user_id
* post_url
* post_type
* created_on
* updated_on

Clustering key created_on desc
Partition by `user_id`. Why?

We can fetch all posts associated with the user from a single db. This will allow user to view his own posts or even when someone searches for a particular user.

## followers table

* followee_id - a person who is being followed.
* follower_id - a person who follows
* created_on - 
* updated_on

Clustering key `created_on desc`.
Partition by `followee_id`. Why?

We can fetch all followers of a user in a single query.


## timeline table

* user_id
* post_id
* created_on

Clustering key `created on desc`.
Partitioning key `user_id`. Why?

We can get the the timeline of a user in a single query

## How to create timeline for a user

1. `user_1` creates a post
1. `user_1` has 2 followers: `user_10` and `user_20`. Now, if `user_10` or `user_20`, open the app they should see `user_1`'s post.
1. Once the posts is saved to db, get the list of followers.
1. Add a row for each follower to the `timeline` table.

When `user_10` opens the app, we fetch the rows from timeline table.

This method works only when you have small number of followers. If a user has millions of followers then this method will cause millions of writes to db, which is not good.

So what we a influencer creates a new post, we don't fetch the followers and update `timeline` table. Instead, we save the influencer post to `posts` table and then to `influencer_posts`.

`influencer_posts` table

* influencer_id
* post_id
* created_on

Clustering key `created_on desc`
Partition by `influencer_id`. Why?  We can fetch influencer posts by `user_id` in a single query.

Now, lets whats happen when a user who follows influencers and non-influencers  tries to view timeline.

For non-influencer posts, we can query the `timeline` table directly.

Next, we fetch all `user_id`s of influencers. Next, we group, the influencers.

Group influencers into 2 groups (we can adjust it):

```text
Group A = infl_1, infl_2, infl_3
Group B = infl_4, infl_5, infl_6
```

Fetch 5 posts from each influencer in group and merge the post group-wise:

```text
Group A = [post_1, post_2, ... ]
Group B = [post_11, post_22, ... ]
```

Merge posts in group A and group B. Also take out the timestamp of the last post and sent it as response (cursor). This cursor allows us to read the second page of the results

For influencer posts, we can query the `timeline` table directly.