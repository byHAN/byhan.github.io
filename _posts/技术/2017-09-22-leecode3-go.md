---
layout: post
title: 3-go
category: leecode
tags: 
keywords: 
description: 
---


Given a string, find the length of the longest substring without repeating characters.

Examples:

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a substring, "pwke" is a subsequence and not a substring.


    func lengthOfLongestSubstring(s string) int {
    	if 0==len(s){
    		return 0
    	}
    	var head int
    	var tail int
    	for i:=0;i<len(s);i++{
    		for j:=i+1;j<len(s);j++{
    			if indexofString(s[i:j],int32(s[j]))>=0{
    				break
    			}else{
    			if j - i >= tail-head+1{
    				head=i
    				tail=j
    				//fmt.Println(s[before:last+1])
    				}
    			}
    		}
    	}
    	//fmt.Println(s[before:last+1])
    	return tail-head+1
    }
    func indexofString( s string,i int32) int {
    	for index1,value1:=range s{
    		if i==value1{
    			return index1
    		}
    	}
    	return -1
    }