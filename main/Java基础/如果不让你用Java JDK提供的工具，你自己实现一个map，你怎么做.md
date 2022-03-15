# 如果不让你用Java JDK提供的工具，你自己实现一个map，你怎么做

```java
public class MyHashMap {
	private final Integer DEFAULTLENGTH = 8;
	private LinkedList[] arr;
	public MyHashMap() {
		arr = new LinkedList[DEFAULTLENGTH];
	}
	public MyHashMap(int n) {
		arr = new LinkedList[n];
	}
	public void put(Object k, Object v) {
		if(k==null) k=0;
		Node node = new Node(k, v);
		int index = Math.abs(k.hashCode()) % arr.length;
		if (arr[index] == null) {
			LinkedList<Node> ln = new LinkedList<Node>();
			arr[index] = ln;
			ln.add(node);
		} else {
			LinkedList<Node> list = arr[index];
			boolean exist = false;
			for (int i = 0; i < list.size(); i++) {
				Node node2 = list.get(i);
				if (node2.getKey().equals(k)) {
					node2.setValue(v);
					exist = true;
					break;
				}
			}
			if (!exist) {
				list.add(node);
			}
		}

	}
	public Object get(Object k) {
		if(k==null) k = 0;
		int index = Math.abs(k.hashCode()) % arr.length;
		if (arr[index] == null) {
			return null;
		} else {
			LinkedList<Node> li = arr[index];
			for (int y = 0; y < li.size(); y++) {
				Node node = li.get(y);
				if (node.getKey().equals(k)) {
					return node.getValue();
				}
			}
			return null;
		}
	}
	private class Node {
		private Object k;
		private Object v;
		Node(Object k, Object v) {
			this.k = k;
			this.v = v;
		}
		Object getKey() {
			return this.k;
		}
		Object getValue() {
			return this.v;
		}
		void setKey(Object k) {
			this.k = k;
		}
		void setValue(Object v) {
			this.v = v;
		}
	}
}
```