在主题文件functions.php中添加如下代码

```
<?php
// 禁用WordPress原生所有自动裁剪图片尺寸
function shapeSpace_disable_image_sizes($sizes) {
    unset($sizes['thumbnail']);
    unset($sizes['medium']);
    unset($sizes['large']);
    unset($sizes['medium_large']);
    unset($sizes['1536x1536']);
    unset($sizes['2048x2048']);
    return $sizes;
}
add_filter('intermediate_image_sizes_advanced', 'shapeSpace_disable_image_sizes');

// 关闭大图自动生成 -scaled 缩放副本
add_filter('big_image_size_threshold', '__return_false');

// 移除特色图片尺寸（需要特色图就删掉这块）
function shapeSpace_disable_other_image_sizes() {
    remove_image_size('post-thumbnail');
}
add_action('init', 'shapeSpace_disable_other_image_sizes');
```