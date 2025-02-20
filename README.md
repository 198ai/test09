Glide.with(cardView.context)
    .load(imageItem.url)
    .skipMemoryCache(true)
    .diskCacheStrategy(DiskCacheStrategy.AUTOMATIC)
    .into(object : CustomTarget<Bitmap>() { // 无尺寸限制
        override fun onResourceReady(resource: Bitmap, transition: Transition<in Bitmap>?) {
            cardView.mainImageView.setImageBitmap(resource)
            val newItem = ImageItem(imageItem.url, WeakReference(resource))
            val index = adapter.indexOf(imageItem)
            if (index >= 0) adapter.set(index, newItem)
        }
        override fun onLoadCleared(placeholder: Drawable?) {
            cardView.setMainImage(null)
        }
    })
