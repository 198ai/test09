class YourVerticalGridFragment : VerticalGridSupportFragment() {
    private val columns = 5 // 列数
    private var lastFirstVisible = -1
    private var lastLastVisible = -1

    // 获取绑定的ArrayObjectAdapter
    private val gridAdapter: ArrayObjectAdapter?
        get() = (gridPresenter as? VerticalGridPresenter)?.adapter as? ArrayObjectAdapter

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
        val adapter = gridAdapter ?: return
        val itemCount = adapter.size() // 关键修正点：从Adapter获取总数

        val layoutManager = recyclerView.layoutManager as LinearLayoutManager
        val firstVisible = layoutManager.findFirstVisibleItemPosition()
        val lastVisible = layoutManager.findLastVisibleItemPosition()

        if (abs(firstVisible - lastFirstVisible) < columns * 3 &&
            abs(lastVisible - lastLastVisible) < columns * 3) return

        val keepStart = (firstVisible - columns * 5).coerceAtLeast(0)
        val keepEnd = (lastVisible + columns * 5).coerceAtMost(itemCount - 1)

        // 清理屏幕外缓存（优化版）
        (0 until itemCount)
            .filter { it < keepStart || it > keepEnd }
            .forEach { position ->
                // 通过ViewHolder清理特定项
                recyclerView.findViewHolderForAdapterPosition(position)?.let { vh ->
                    (presenter as? MyImagePresenter)?.onUnbindViewHolder(vh)
                }
            }

        lastFirstVisible = firstVisible
        lastLastVisible = lastVisible
    }
}
