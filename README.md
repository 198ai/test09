class YourVerticalGridFragment : VerticalGridSupportFragment() {

    private val columns = 5 // 列数
    private var lastFirstVisible = -1
    private var lastLastVisible = -1

    // 获取绑定的ArrayObjectAdapter
    private val gridAdapter: ArrayObjectAdapter?
        get() = (gridPresenter as? VerticalGridPresenter)?.adapter as? ArrayObjectAdapter

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // 使用 VerticalGridSupportFragment 内置的 verticalGridView，而不是自己 findViewById
        verticalGridView?.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrollStateChanged(rv: RecyclerView, newState: Int) {
                if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                    manageCacheRange(rv)
                }
            }
        })
    }

    private fun manageCacheRange(recyclerView: RecyclerView) {
        val adapter = gridAdapter ?: return
        val itemCount = adapter.size() // 必须从 Adapter 获取总数

        // 注意：verticalGridView 实际继承的是 BaseGridView
        val gridView = recyclerView as? BaseGridView ?: return

        val childCount = gridView.childCount
        if (childCount == 0) return

        // 第一个可见子 View
        val firstView = gridView.getChildAt(0) ?: return
        // 最后一个可见子 View
        val lastView = gridView.getChildAt(childCount - 1) ?: return

        // 通过 getChildAdapterPosition(...) 获取对应的 Adapter 索引
        val firstVisible = gridView.getChildAdapterPosition(firstView)
        val lastVisible = gridView.getChildAdapterPosition(lastView)

        // 若计算到无效的 position（比如 -1），直接返回
        if (firstVisible < 0 || lastVisible < 0) return

        // 防止频繁刷新，只有在滚动超过一定距离后再执行清理
        if (abs(firstVisible - lastFirstVisible) < columns * 3 &&
            abs(lastVisible - lastLastVisible) < columns * 3
        ) return

        // 计算要保留的起始和结束范围
        val keepStart = (firstVisible - columns * 5).coerceAtLeast(0)
        val keepEnd = (lastVisible + columns * 5).coerceAtMost(itemCount - 1)

        // 清理屏幕外缓存
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
