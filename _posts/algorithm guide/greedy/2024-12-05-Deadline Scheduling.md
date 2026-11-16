---
title: Deadline Scheduling Problem
date: 2024-12-05 16:00:00 +0900
categories: [algorithm guide, greedy]
tags: [cs, greedy]
---

Problem statement:

- You’re given $n$ jobs that have a certain deadline and the time it takes to complete the job
- You need to optimally schedule the job and find the maximal number of jobs you can take
- Note here, the deadline is when the task should END.

Approach:

- Sort all the jobs by deadline, and let $t$ be the current time
- Let’s iterate through the jobs
    - We put it in our max heap
    - Now, let’s add the duration to $t$
    - If $t$ is larger than the current deadline, that means we can’t take the task
        - We pop the largest duration task from our max heap until we satisfy our condition
        - But, this can only be up to one job. Thus, we simply pop once
- The final size of our heap is the maximal number of jobs. The content of our max heap represents the individual optimal jobs

[n4435898]

```cpp
sort(jobs.begin(), jobs.end());

ll currTime = 0;
priority_queue<ll> pq;
for(int i=0;i<n;i++){
    currTime += jobs[i].second;
    pq.push(jobs[i].second);
    if(currTime > jobs[i].first){
        currTime -= pq.top();
        pq.pop();
    }
}
cout<<pq.size();
```

[CP-Algo]

```cpp
struct Job {
    int deadline, duration, idx;

    bool operator<(Job o) const {
        return deadline < o.deadline;
    }
};

vector<int> compute_schedule(vector<Job> jobs) {
    sort(jobs.begin(), jobs.end());

    set<pair<int,int>> s;
    vector<int> schedule;
    for (int i = jobs.size()-1; i >= 0; i--) {
        int t = jobs[i].deadline - (i ? jobs[i-1].deadline : 0);
        s.insert(make_pair(jobs[i].duration, jobs[i].idx));
        while (t && !s.empty()) {
            auto it = s.begin();
            if (it->first <= t) {
                t -= it->first;
                schedule.push_back(it->second);
            } else {
                s.insert(make_pair(it->first - t, it->second));
                t = 0;
            }
            s.erase(it);
        }
    }
    return schedule;
}
```
