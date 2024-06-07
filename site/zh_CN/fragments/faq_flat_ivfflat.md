<p>IVF_FLAT 索引将一个向量空间划分为 <code>nlist</code> 个簇。如果保持 <code>nlist</code> 的默认值为 16384，Milvus 将比较目标向量与所有 16384 个簇中心点之间的距离，以获取 <code>nprobe</code> 个最近的簇。然后 Milvus 比较目标向量与所选簇中的向量之间的距离，以获取最近的向量。与 IVF_FLAT 不同，FLAT 直接比较目标向量与每个向量之间的距离。</p>
<p>
因此，当向量总数大约等于 <code>nlist</code> 时，IVF_FLAT 和 FLAT 在所需计算方式和搜索性能方面几乎没有差异。但是，当向量数量增长到 <code>nlist</code> 的两倍、三倍或 n 倍时，IVF_FLAT 索引开始显示出越来越大的优势。</p>
<p>
更多信息请参阅 <a href="https://medium.com/unstructured-data-service/how-to-choose-an-index-in-milvus-4f3d15259212">How to Choose an Index in Milvus</a>。</p>