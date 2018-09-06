---
layout: page
title: Playground
tags: [MS CS, IITB, NYU, Computer Science]
comments: false
---
### NYC Blockchain Developer Meetup
Some of the example contracts written in solidity during weekly meeting of ethereum study group. 
Please find more about this meetup at [link]({{"https://www.meetup.com/nyc-blockchain-devs/"}}) 
1. [Decentralized Blackjack]({{"https://github.com/vutsalsinghal/decentralised_BlackJack"}})
2. 2-Party Betting Contract and Joint Account Contract. [github]({{https://github.com/panghalamit/smart_contracts}})

### Sample Codes for interesting leetcode questions

1. Given an array nums, we call (i, j) an important reverse pair if i < j and nums[i] > 2*nums[j]. You need to return the number of important reverse pairs in the given array.

{% highlight cpp %}
  int reversePairsHelper(vector<int>& nums, vector<int>& nums_extra, int L, int U) {
        if(U > L) {
            int result=0, result_temp=0;
            int mid = (U+L)/2;
            result += reversePairsHelper(nums_extra, nums, L, mid);
            result += reversePairsHelper(nums_extra, nums, mid+1, U);
            int r1 = L, r2 = mid+1, r3 = L;
            while(r1 <= mid && r2 <= U) {
                if(nums_extra[r1]/2.0 > nums_extra[r2]*1.0 ) {
                    result_temp++;
                    r2++;
                } else {
                    result += result_temp;
                    r1++;
                }
            }
            while(r1<=mid) {
                result+=result_temp;
                r1++;
            }
            r1=L; r2=mid+1;r3=L;
            while(r1 <= mid && r2 <= U) {
                if(nums_extra[r1] > nums_extra[r2]) {
                    nums[r3++] = nums_extra[r2++];
                } else {
                    nums[r3++] = nums_extra[r1++];

                }
            }
            while(r1 <= mid) {
                nums[r3++] = nums_extra[r1++];
            }
            while(r2 <= U) {
                nums[r3++] = nums_extra[r2++];
            }
            return result;
        }
        else return 0;
    }

    int reversePairs(vector<int>& nums) {
        vector<int> nums_extra(nums);
        return reversePairsHelper(nums, nums_extra,0, nums.size()-1);
    }
{% endhighlight %}
