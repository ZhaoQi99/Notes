## 实现
```python title:celery.py
def revoke(celery_id):
    x = AsyncResult(celery_id)
    x.revoke(terminate=True)
    if x.successful():
        group = x.children[0]
        children = list()
        if isinstance(group, GroupResult):  
            children = group.children # GroupResult -> [AsyncResult]
        elif isinstance(group, AsyncResult):
            children = [group]

        for child in children:
            child.revoke(terminate=True)
        else:
            AsyncResult(x.result[0][0]).revoke(terminate=True)
```
## 示例
```python title:test.py
from celery import group, chain, shared_task

@shared_task
def add(x, y):
    return x + y

@shared_task
def summarize(results):
    return sum(results)

@shared_task
def test_workflow(num=2):
	# sub_tasks = group(add.s(i, i) for i in range(num))
	# workflow = chain(sub_tasks, summarize.s())()
	workflow = chord(
	    [add.s(i, i) for i in range(num)],
	    summarize.s(),
	)()
	print(f"Workflow ID: {workflow.id}")
	return workflow
```
## Note
* `chord` 返回值是**回调任务的 `AsyncResult` (`workflow.id` 是 `summarize.s()` 任务的 ID)
* `group` 只有 1 个子任务时,也会升级为 chord,但是callback 的参数和多子任务时不同 (非 list, 为单个值)

## 参考文档
* [Canvas: Designing Work-flows — Celery 5.6.2 documentation](https://docs.celeryq.dev/en/stable/userguide/canvas.html#chords)