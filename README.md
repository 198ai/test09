class YourVerticalGridFragment : VerticalGridSupportFragment() {
    private val columns = 5 // 你的列数
    private var lastFirstVisible = -1
    private var lastLastVisible = -1
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        view.findViewById<RecyclerView>(androidx.leanback.R.id.browse_grid_dump)?.apply {
            addOnScrollListener(object : RecyclerView.OnScrollListener() {
                override fun onScrollStateChanged(recyclerView: RecyclerView, newState: Int) {
                    if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                        manageCacheRange(recyclerView)
                    }
                }
            })
        }
    }

    private fun manageCacheRange(recyclerView: RecyclerView) {
        val layoutManager = recyclerView.layoutManager as LinearLayoutManager
        val firstVisible = layoutManager.findFirstVisibleItemPosition()
        val lastVisible = layoutManager.findLastVisibleItemPosition()
        
        // 如果滚动范围变化小于3行则跳过
        if (abs(firstVisible - lastFirstVisible) < columns * 3 &&
            abs(lastVisible - lastLastVisible) < columns * 3) return

        val keepStart = (firstVisible - columns * 5).coerceAtLeast(0)
        val keepEnd = (lastVisible + columns * 5).coerceAtMost(itemCount - 1)
        
        // 清除屏幕外缓存
        (0 until itemCount).filter { it < keepStart || it > keepEnd }
            .mapNotNull { getItem(it) as? YourDataItem }
            .forEach { item ->
                Glide.get(requireContext()).clearMemory()
                // 可选：清除磁盘缓存（需要异步处理）
                // lifecycleScope.launch(Dispatchers.IO) {
                //     Glide.get(requireContext()).clearDiskCache()
                // }
            }

        lastFirstVisible = firstVisible
        lastLastVisible = lastVisible
    }
}
