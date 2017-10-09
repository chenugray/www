package difflib;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Test {
	public static void main(String[] args) throws Exception {
		List<String> origin = new ArrayList<String>();
		List<String> revised = new ArrayList<String>();
		File originFile = new File("/Users/_why/Desktop/1.txt");
		File reviseFile = new File("/Users/_why/Desktop/2.txt");

		BufferedReader reader1 = new BufferedReader(new FileReader(originFile));
		BufferedReader reader2 = new BufferedReader(new FileReader(reviseFile));
		String tmp = null;
		while ((tmp = reader1.readLine()) != null) {
			origin.add(tmp);
		}
		while ((tmp = reader2.readLine()) != null) {
			revised.add(tmp);
		}
		Patch<String> p = DiffUtils.diff(revised, origin);
		List<Delta<String>> dts = p.getDeltas();
		for (Delta<String> delta : dts) {
			System.out.println(delta);
		}
		List<Change> changes = new ArrayList<Change>();
		int offset = 0;

		for (Delta<String> delta : dts) {
			if (delta instanceof InsertDelta) {
				int position = delta.getOriginal().getPosition();
				List<String> lines = delta.getRevised().getLines();
				for (int i = 0; i < lines.size(); i++) {
					revised.add(position + i + offset, "--" + lines.get(i));
				}
				offset++;
			} else if (delta instanceof DeleteDelta) {
				int position = delta.getOriginal().getPosition();
				int size = delta.getOriginal().size();
				for (int i = 0; i < size; i++) {
					String removed = revised.remove(position + offset);
					Change c = new Change(position + offset, "++" + removed);
					changes.add(c);
					// System.out.println("now" + revised);
					// System.out.println("--" + c.getIndex() + " value" +
					// c.getValue());

				}
				offset -= size;
			} else {
				int position = delta.getOriginal().getPosition();
				int size = delta.getOriginal().size();
				for (int i = 0; i < size; i++) {
					String removed = revised.remove(position + offset);
					Change c = new Change(position + offset, "^^" + removed);
					changes.add(c);
					System.out.println("now" + revised);
					System.out.println("--" + c.getIndex() + " value" + c.getValue());
				}

				int i = 0;
				for (String line : delta.getRevised().getLines()) {
					revised.add(position + i + offset, "**" + line);
					i++;

				}
				offset += delta.getRevised().getLines().size();
				offset -= size;
			}
		}

		for (Change change : changes) {
			System.out.println("insert at: " + change.getIndex() + " value:" + change.getValue());
		}
		Collections.reverse(changes);
		for (Change change : changes) {
			revised.add(change.getIndex(), change.getValue());
		}
		System.out.println(revised);
	}
}

class Change {
	private int index;
	private String value;

	public Change(int index, String value) {
		this.index = index;
		this.value = value;
	}

	public int getIndex() {
		return index;
	}

	public void setIndex(int index) {
		this.index = index;
	}

	public String getValue() {
		return value;
	}

	public void setValue(String value) {
		this.value = value;
	}

}
