---
layout:     post
title:      "Sorting algorithm"
author:     "Iceberg"
catalog:    true
header-img: "assets/images/header/yellowstone7.jpg"
tags:
  - Algorithm
  - Programming
---

In this post, it includes the following sorting algorithms.

* selection sort
* insert sort
* bubble sort
* quick sort
* merge sort

```java
import java.util.Arrays;

public class Sort {
	public static void selection_sort(int[] arr) {
		for (int i = 0; i < arr.length - 1; i++) {
			int min_idx = i;

			// find the minimum value from right of arr[i] and swap with arr[i]
			for (int j = i + 1; j < arr.length; j++) {
				if (arr[j] < arr[min_idx]) {
					min_idx = j;
				}
			}

			// move the min value to the beginning of array
			int temp = arr[i];
			arr[i] = arr[min_idx];
			arr[min_idx] = temp;
		}
	}

	public static void insert_sort(int[] arr) {
		// sort from the second element since the first one is already sorted for itself
		for (int i = 1; i < arr.length; i++) {
			int curr = arr[i];
			int j = i - 1;

			// move all the greater values(than curr) to one position right
			while (j >= 0 && arr[j] > curr) {
				arr[j + 1] = arr[j];
				j--;
			}

			arr[j + 1] = curr;
		}
	}

	public static void bubble_sort(int[] arr) {
		// bubble sort for n - 1 rounds
		for (int i = 0; i < arr.length - 1; i++) {
			boolean swapped = false;
			// For each round, bubble up the maximum element to the right
			for (int j = 0; j < arr.length - i - 1; j++) {
				if (arr[j] > arr[j + 1]) {
					int temp = arr[j];
					arr[j] = arr[j + 1];
					arr[j + 1] = temp;
					swapped = true;
				}
			}

			if (!swapped)
				break;
		}
	}

	public static void quick_sort(int[] arr, int left, int right) {
		if (left < right) {
			int pivot = partition(arr, left, right);
			quick_sort(arr, left, pivot - 1);
			quick_sort(arr, pivot + 1, right);
		}
	}

	private static int partition(int[] arr, int left, int right) {
		int pivot = arr[right];
		int curr = left - 1;

		for (int i = left; i < right; i++) {
			if (arr[i] <= pivot) {
				curr++;
				int temp = arr[curr];
				arr[curr] = arr[i];
				arr[i] = temp;
			}
		}

		curr++;
		int temp = arr[curr];
		arr[curr] = pivot;
		arr[right] = temp;
		return curr;
	}

	public static void merge_sort(int[] arr, int left, int right) {
		if (left < right) {
			//int mid = (left + right) / 2;
			int mid = left + (right - left) / 2; // avoid overflow
			merge_sort(arr, left, mid);
			merge_sort(arr, mid + 1, right);
			merge(arr, left, mid, right);
		}
	}

	private static void merge(int[] arr, int left, int mid, int right) {
		int l1 = mid - left + 1;
		int l2 = right - mid;

		// copy left and right to the temporary array
		int[] a1 = new int[l1];
		int[] a2 = new int[l2];


		for (int i = 0; i < l1; i++) {
			a1[i] = arr[left + i];
		}

		for (int i = 0; i < l2; i++) {
			a2[i] = arr[mid + 1 + i];
		}

		// merge back to the original array
		int i = 0, j = 0, k = left;
		while (i < l1 && j < l2) {
			if (a1[i] <= a2[j]) {
				arr[k++] = a1[i++];
			} else {
				arr[k++] = a2[j++];
			}
		}

		while (i < l1) {
			arr[k++] = a1[i++];
		}
		while (j < l2) {
			arr[k++] = a2[j++];
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] arr = { 11, 25, 12, 22, 64 };
		selection_sort(arr);
		System.out.println(Arrays.toString(arr));

		int[] arr1 = { 11, 25, 12, 22, 64 };
		insert_sort(arr1);
		System.out.println(Arrays.toString(arr1));

		int[] arr2 = { 11, 25, 12, 22, 64 };
		bubble_sort(arr2);
		System.out.println(Arrays.toString(arr2));

		int[] arr3 = { 11, 25, 12, 22, 64 };
		quick_sort(arr3, 0, arr3.length - 1);
		System.out.println(Arrays.toString(arr3));

		int[] arr4 = { 11, 25, 12, 22, 64 };
		merge_sort(arr4, 0, arr4.length - 1);
		System.out.println(Arrays.toString(arr4));
	}

}
```