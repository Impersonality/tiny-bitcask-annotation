1.tiny-bit和minidb的merge不同，tiny-bit不合并active_file，只合并inactive_file，遍历inactive中的entry,如果和index中的
offset和fid相同（该key最新数据），则写入到active_file中，然后删除inactive_file，该过程实质是压缩所有file到active_file。
minidb只有一个file，merge的过程是new一个file，然后遍历entry，和index中offset相同的写入的new_file，然后删除原file。

2.tiny-bit的storage(对应minidb的db_file)有两个read_entry方法，一个是readEntry(fid int, off int64) (e *Entry, err error)，
一个是readFullEntry(fid int, off int64, buf []byte) (e *Entry, err error)，db.Get使用readFullEntry，merge/recovery
使用一个是readEntry，原因是readFullEntry的使用条件是知道key_size和val_size，也就是有索引的时候使用。readFullEntry的优点
是与磁盘一次交互读取entry，不用先获得meta_data，然后再根据key_size和val_size第二次读写磁盘。

3.tiny-bit的index主要存储有offset和fid，minidb是offset，因为tiny-bit有active_file结构。tiny-bit的index还有key_size和
val_size，上述第二点提到了，index存储key_size，db.Get读取entry时可以一次磁盘读写获得entry。

4.上面3点看似在表示tiny-bit比minidb多一些优点，其实是tiny-bit像他的仓库名一样，实现的是bitcask结构，minidb的实现是一个最基本的
用于学习的kv db，minidb的代码可读性很高，我认为算的上优美。比如tiny-bit storage中读取entry，需要new一个entry作为载体，然后decode
buf，minidb直接decode 到entry，tiny-bit 的new Entry函数是有迷惑性的，这应该算代码审美吧，我需要多多学习。