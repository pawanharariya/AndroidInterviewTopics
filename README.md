# Android Interview Topics
## Recycler View Optimisation Techniques ##
1. DiffUtils
2. setHasFixedSize
3. notifyItemChanged() methods
4. Using notifyItemChanged(payload : List<Any>()) - inside BindViewHolder check if payload is empty else use payload to update instead of redrawing everything.
5. ListLiveData implementation - [Read more here](https://medium.com/google-developer-experts/notifying-recyclerview-on-a-specific-change-b36e6dc59e0f)

