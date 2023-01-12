---
slug: entity-interpolation
title: Nội suy thực thể
authors: [anh]
tags: [multiplayer, sync entity, game dev, interpolation]
---

Một vấn đề mình gặp phải trong quá trình học làm mutiplayer.

# Vấn đề

Các nhân vật khác với client đang điều khiển sẽ lấy dữ liệu vị trí từ server để đồng bộ.
Ban đầu mình sử dùng một hàm kiểu như này:

```c# title="SyncViewWithServer()"
Vector3 targetPos = state.position; // get last position from server
var step =  speed * Time.deltaTime; // calculate distance to move
transform.position = Vector3.MoveTowards(transform.position, targetPos, step);
```

Dùng hàm MoveTowards mục đích để di chuyển mượt mà từ vị trí hiện tại đến vị trí lấy được từ server (Có thể dùng hàm Lerp hoặc SmoothDamp).
Thực tế nó lại không như mình tưởng tượng, nhân vật di chuyển giật giật.

# Giải pháp

ok

# Entity interpolation
