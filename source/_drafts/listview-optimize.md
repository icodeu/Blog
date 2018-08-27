title: listview-optimize

## tags: Android

打算写个系列文章好好说一下 ListView 的优化问题（或GridView，其实差不了多少）。



如果 ListView 里只有文字，其实对于内存占用方面来说也没有什么可优化的，滚动起来也不会明显卡顿，关键就在于如果 ListView 中有图片了，这个时候就要好好想想需要怎么来进行优化了，所以本系列文章就分层次分别说说 ListView 图片加载优化问题，包含两个层次：ListView 本身的优化问题，图片的内存加载问题。



## ListView 本身的优化

1、一定要利用好 convertView，减少 inflate 布局的时间

2、使用 ViewHolder，减少 findViewById 的时间

3、滚动不加载、停止才加载，按需加载，减少滑动卡顿



## 图片的内存加载优化

1、减小图片本身占用内存大小

2、防止图片错位

3、使用内存缓存

4、使用外存缓存