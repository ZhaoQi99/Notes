* Usage: `python manage.py runscript status_worker`

```python title:<app>/scripts/stats_worker.py
from django.conf import settings

from <project_name>.celery import app

ALL_QUEUE_NUM = len(settings.CELERY_TASK_QUEUES)
ALL_QUEUE_NAMES = {queue.name for queue in settings.CELERY_TASK_QUEUES}


def run():
    inspect = app.control.inspect()
    res = {"all": []}

    active_queues = (inspect.active_queues() or {}).items()

    for worker_name, info in active_queues:
        if len(info) == ALL_QUEUE_NUM:
            res["all"].append(worker_name)
            continue

        for queue in info:
            queue_name = queue["name"]
            res.setdefault(queue_name, list())
            res[queue_name].append(worker_name)

    worker_num = len(active_queues)
    queue_num = len(res) - 1

    print("Celery Status Report")
    print("=====================")
    print(f"Total Workers: {worker_num}, Total Queues: {queue_num}")

    not_declared = ALL_QUEUE_NAMES - set(res.keys())
    print(f"Undeclared Queues: {', '.join(not_declared)}")
    print()

    print("Queue Worker Distribution")
    print("-------------------------")
    for queue, workers in res.items():
        print(f"{queue.capitalize()} Queue ({len(workers)} workers):")
        print("\n".join([f"  - {worker}" for worker in workers]))
        print()


```
Examples:
```shell
~$ python manage.py runscript status_worker
Celery Status Report
=====================
Total Workers: 4, Total Queues: 4
Undeclared Queues: C, D

Queue Worker Distribution
-------------------------
All Queue (0 workers):

A Queue (2 workers):
  - worker1
  - worker2

B Queue (2 workers):
  - worker3
  - worker4
```
