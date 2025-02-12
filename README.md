class YourVerticalGridFragment : VerticalGridSupportFragment() {
    private val columns = 5
    private var lastSelectedRow = -1
    private var selectedPositionIndex = 0

    // 添加选中位置监听
    init {
        onItemViewSelectedListener = OnItemViewSelectedListener { _, _, _, position ->
            selectedPositionIndex = position
        }
    }

    private val gridAdapter: ArrayObjectAdapter?
        get() = (gridPresenter as? VerticalGridPresenter)?.adapter as? ArrayObjectAdapter

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
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
        val itemCount = adapter.size()
        if (itemCount == 0) return

        val currentRow = selectedPositionIndex / columns
        val totalRows = (itemCount + columns - 1) / columns // 计算总行数（向上取整）

        // 确定滚动方向
        val direction = when {
            currentRow > lastSelectedRow -> Direction.DOWN
            currentRow < lastSelectedRow -> Direction.UP
            else -> null
        }
        lastSelectedRow = currentRow

        // 需要清理的位置范围
        val positionsToClear = mutableListOf<Int>()

        when (direction) {
            Direction.DOWN -> {
                // 如果当前行上方超过 10 行，清理最前面 5 行
                if (currentRow > 10) {
                    val clearEndRow = 4 // 清理前 5 行（0-4）
                    val clearEndPos = min((clearEndRow + 1) * columns, itemCount) - 1
                    positionsToClear.addAll(0..clearEndPos)
                }
            }
            Direction.UP -> {
                // 如果下方剩余超过 10 行，清理最后 5 行
                val remainingRows = totalRows - currentRow - 1
                if (remainingRows > 10) {
                    val clearStartRow = max(totalRows - 5, 0)
                    val clearStartPos = clearStartRow * columns
                    positionsToClear.addAll(clearStartPos until itemCount)
                }
            }
            null -> return
        }

        // 清理图片并打印日志
        positionsToClear.forEach { position ->
            recyclerView.findViewHolderForAdapterPosition(position)?.let { vh ->
                val cardView = vh as? BasContentImageCardView
                cardView?.let {
                    // 获取 URL（需要根据你的数据结构实现）
                    val url = getUrlFromPosition(position)
                    println("Cleared image at position: $position, URL: $url")
                    
                    Glide.with(it.context).clear(it.imageView)
                }
            }
        }
    }

    // 示例方法：从数据源获取 URL（需要根据实际数据结构调整）
    private fun getUrlFromPosition(position: Int): String? {
        return (gridAdapter?.get(position) as? YourDataItem)?.imageUrl
    }

    enum class Direction { UP, DOWN }
}
