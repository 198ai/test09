class YourVerticalGridFragment : VerticalGridSupportFragment() {
    private val columns = 5
    private var lastSelectedRow = -1
    private var selectedPositionIndex = 0
    private val visiblePositions = mutableSetOf<Int>()

    // 维护需要保留的图片范围
    private var keepStart = 0
    private var keepEnd = 0

    init {
        onItemViewSelectedListener = OnItemViewSelectedListener { _, _, _, position ->
            selectedPositionIndex = position
        }
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        verticalGridView?.addOnScrollListener(object : RecyclerView.OnScrollListener() {
            override fun onScrollStateChanged(rv: RecyclerView, newState: Int) {
                if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                    updateVisibleRange()
                    manageCacheRange()
                }
            }
        })
    }

    private fun updateVisibleRange() {
        visiblePositions.clear()
        verticalGridView?.apply {
            for (i in 0 until childCount) {
                getChildAdapterPosition(getChildAt(i))?.takeIf { it != RecyclerView.NO_POSITION }?.let {
                    visiblePositions.add(it)
                }
            }
        }
    }

    private fun manageCacheRange() {
        val adapter = gridAdapter ?: return
        val itemCount = adapter.size()
        if (itemCount == 0) return

        val currentRow = selectedPositionIndex / columns
        val totalRows = ceil(itemCount.toDouble() / columns).toInt()

        // 计算需要保留的边界（扩展5屏范围）
        val newKeepStart = ((currentRow - 5).coerceAtLeast(0) * columns).also { keepStart = it }
        val newKeepEnd = ((currentRow + 5).coerceAtMost(totalRows - 1) * columns + columns).coerceAtMost(itemCount).also { keepEnd = it }

        // 打印保留范围用于调试
        println("Keeping positions: $newKeepStart - $newKeepEnd")

        // 遍历所有可能的位置清理缓存
        (0 until itemCount).filter { it !in newKeepStart..newKeepEnd }.forEach { position ->
            // 只有当位置不在可见区域时才清理（避免闪烁）
            if (position !in visiblePositions) {
                clearImageForPosition(position)
            }
        }
    }

    private fun clearImageForPosition(position: Int) {
        // 从数据源获取URL（需要根据实际数据结构调整）
        val url = (gridAdapter?.get(position) as? YourDataItem)?.imageUrl

        // 使用Glide的缓存API直接清理
        Glide.get(requireContext()).apply {
            // 清理内存缓存
            clearMemory()
            
            // 异步清理磁盘缓存（可选）
            Thread {
                clearDiskCache()
            }.start()
        }

        // 打印清理日志
        println("Cleared cache for position: $position, URL: ${url ?: "unknown"}")
    }
}
