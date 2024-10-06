检查文件是否存在
```
Path path = Paths.get("helloword/data.txt");
System.out.println(Files.exists(path));
```
创建一级目录
```
Path path = Paths.get("helloword/d1");
Files.createDirectory(path);
```
● 如果目录已存在，会抛异常 FileAlreadyExistsException
● 不能一次创建多级目录，否则会抛异常 NoSuchFileException
创建多级目录用
```
Path path = Paths.get("helloword/d1/d2");
Files.createDirectories(path);
```
拷贝文件
```
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/target.txt");
Files.copy(source, target);
```
● 如果文件已存在，会抛异常 FileAlreadyExistsException
如果希望用 source 覆盖掉 target，需要用 StandardCopyOption 来控制
```
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
移动文件
Path source = Paths.get("helloword/data.txt");
Path target = Paths.get("helloword/data.txt");
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
● StandardCopyOption.ATOMIC_MOVE 保证文件移动的原子性
```
删除文件
```
Path target = Paths.get("helloword/target.txt");
Files.delete(target);
```
● 如果文件不存在，会抛异常 NoSuchFileException
删除目录
```
Path target = Paths.get("helloword/d1");
Files.delete(target);
```
● 如果目录还有内容，会抛异常 DirectoryNotEmptyException
遍历目录文件
```
public static void main(String[] args) throws IOException {
	Path path = Paths.get("C:\\Program Files\\Java\\jdk1.8.0_91");
	AtomicInteger dirCount = new AtomicInteger();
	AtomicInteger fileCount = new AtomicInteger();
	Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
		@Override
		public FileVisitResult preVisitDirectory(Path dir,BsicFileAttributes attrs)
			throws IOException {
			System.out.println(dir);
			dirCount.incrementAndGet();
			return super.preVisitDirectory(dir, attrs);
		}
		@Override
		public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)
			throws IOException {
			System.out.println(file);
			fileCount.incrementAndGet();
			return super.visitFile(file, attrs);
		}
	});
	System.out.println(dirCount); // 133
	System.out.println(fileCount); // 1479
}
```
统计 jar 的数目
```
Path path = Paths.get("C:\\Program Files\\Java\\jdk1.8.0_91");
AtomicInteger fileCount = new AtomicInteger();
Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)
		throws IOException {
		if (file.toFile().getName().endsWith(".jar")) {
		fileCount.incrementAndGet();
		}
	return super.visitFile(file, attrs);
	}
});
System.out.println(fileCount); // 724
```
删除多级目录
```
Path path = Paths.get("d:\\a");
Files.walkFileTree(path, new SimpleFileVisitor<Path>(){
	@Override
	public FileVisitResult visitFile(Path file, BasicFileAttributes attrs)throws IOException {
		Files.delete(file);
		return super.visitFile(file, attrs);
	}
	@Override
	public FileVisitResult postVisitDirectory(Path dir, IOException exc)throws IOException {
		Files.delete(dir);
		 return super.postVisitDirectory(dir, exc);
	}
});
```
⚠️ 删除很危险
删除是危险操作，确保要递归删除的文件夹没有重要内容
拷贝多级目录
long start = System.currentTimeMillis();
String source = "D:\\Snipaste-1.16.2-x64";
String target = "D:\\Snipaste-1.16.2-x64aaa";
Files.walk(Paths.get(source)).forEach(path -> {
try {
String targetName = path.toString().replace(source, target);
// 是目录
if (Files.isDirectory(path)) {
Files.createDirectory(Paths.get(targetName));
}
// 是普通文件
else if (Files.isRegularFile(path)) {
Files.copy(path, Paths.get(targetName));
}
} catch (IOException e) {
e.printStackTrace();
 }
});
long end = System.currentTimeMillis();
System.out.println(end - start);