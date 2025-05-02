<template>
  <div class="editor-container">
    <div
      ref="editor"
      class="editor"
      contenteditable="true"
      @input="onEditorInput"
    ></div>
    <p style="display: none">{{ dummYInt }}</p>
    <button @click="insertStyledButton">Insert Draggable Button</button>
    <button @click="triggerImageUpload">Insert Background Image</button>
    <button @click="applyCustomList('ol')">Ordered List</button>
    <button @click="applyCustomList('ul')">Unordered List</button>
    <button @click="insertGrid">Insert Grid</button>
    <button @click="insertLink">Insert Link</button>
    <div class="toolbar">
      <button @click="format('bold')" :class="{ active: isActive('bold') }">
        <b>B</b>
      </button>
      <button @click="format('italic')" :class="{ active: isActive('italic') }">
        <i class="fa-solid fa-italic"></i>
      </button>
      <button
        @click="insertList('unordered')"
        :class="{ active: isActive('insertUnorderedList') }"
      >
        <i class="fa-solid fa-list-ul"></i>Ul
      </button>
      <button
        @click="insertList('ordered')"
        :class="{ active: isActive('insertOrderedList') }"
      >
        <i class="fa-solid fa-list-ol"></i>Ol
      </button>
    </div>

    <input
      ref="fileInput"
      type="file"
      accept="image/*"
      @change="handleImageUpload"
      style="display: none"
    />
    <!-- Modal for editing button styles -->
    <ButtonStylePopup ref="popup" />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, nextTick } from "vue";
import interact from "interactjs";
import ButtonStylePopup from "./ButtonStylePopup.vue"; // Import the popup component

const editor = ref(null);
const popup = ref(null); // Define ref for ButtonStylePopup
const fileInput = ref(null);
const dummYInt = ref(0);
const editorContent = ref(null);
const imageText = ref(null);
const isActive = (command) => {
  const selection = window.getSelection();
  if (!selection.rangeCount) return false;

  const node = selection.getRangeAt(0).startContainer;

  if (!editor.value.contains(node)) return false; // <- ensure it's inside this editor

  let currentNode = node;
  while (currentNode) {
    if (command === "insertUnorderedList" && currentNode.nodeName === "UL") {
      return true;
    }
    if (command === "insertOrderedList" && currentNode.nodeName === "OL") {
      return true;
    }
    currentNode = currentNode.parentNode;
  }

  return document.queryCommandState(command);
};

const updateContent = () => {
  const selection = window.getSelection();
  let range =
    selection && selection.rangeCount > 0 ? selection.getRangeAt(0) : null;

  let markerId = "__cursor_marker__";

  // 1. Insert a temporary marker as text
  if (range) {
    const markerNode = document.createTextNode(markerId);
    range.insertNode(markerNode);
    dummYInt.value++; // Optional: to trigger reactivity
  }

  // 2. Update editorContent with marker
  editorContent.value = editor.value?.innerHTML || "";

  // 3. Restore cursor and clean up
  nextTick(() => {
    const editorEl = editor.value;
    if (!editorEl) return;

    const walker = document.createTreeWalker(
      editorEl,
      NodeFilter.SHOW_TEXT,
      null
    );
    let node,
      offset = 0;
    while ((node = walker.nextNode())) {
      const index = node.nodeValue.indexOf(markerId);
      if (index !== -1) {
        const range = document.createRange();
        const selection = window.getSelection();

        // Set cursor after the marker
        range.setStart(node, index);
        range.collapse(true);

        // Cleanup marker text
        node.nodeValue = node.nodeValue.replace(markerId, "");

        // Restore selection
        selection.removeAllRanges();
        selection.addRange(range);
        break;
      }
    }
  });
};

const format = (command) => {
  const selection = window.getSelection();
  if (!selection.rangeCount) return;

  const range = selection.getRangeAt(0);
  // Only format if selection is inside this editor
  if (!editor.value.contains(range.startContainer)) return;

  document.execCommand(command, false, null);
  //nextTick(() => updateContent());
};

const recreateList = (type) => {
  const selection = window.getSelection();
  if (!selection || selection.rangeCount === 0) return;

  const range = selection.getRangeAt(0);
  if (
    !editor.value?.contains(range.startContainer) &&
    !imageText.value?.contains(range.startContainer)
  )
    return;

  const selectedText = selection.toString().trim();
  if (!selectedText) return;

  const lines = selectedText.split("\n").filter((line) => line.trim() !== "");
  if (!lines.length) return;

  const list = document.createElement(type === "unordered" ? "ul" : "ol");
  lines.forEach((line) => {
    const li = document.createElement("li");
    li.textContent = line.trim();
    list.appendChild(li);
  });

  if (type === "unordered") {
    list.style.listStyleType = "disc";
    list.style.paddingLeft = "2rem";
  }

  range.deleteContents();
  range.insertNode(list);

  const newRange = document.createRange();
  newRange.setStartAfter(list);
  selection.removeAllRanges();
  selection.addRange(newRange);

  nextTick(() => updateContent());
};

const insertList = (type) => {
  const selection = window.getSelection();
  if (!selection || selection.rangeCount === 0) return;

  const range = selection.getRangeAt(0);

  const inEditor = editor.value?.contains(range.startContainer);
  const inImageText = imageText.value?.contains(range.startContainer);
  if (!inEditor && !inImageText) return;

  let node = range.startContainer;
  let insideList = false;
  let foundList = null;

  while (node && node !== editor.value && node !== imageText.value) {
    if (node.nodeName === "UL" || node.nodeName === "OL") {
      insideList = true;
      foundList = node;
      break;
    }
    node = node.parentNode;
  }

  if (insideList && foundList) {
    const fragment = document.createDocumentFragment();
    const children = Array.from(foundList.children);
    let lastInserted = null;

    children.forEach((li) => {
      while (li.firstChild) {
        lastInserted = li.firstChild;
        fragment.appendChild(li.firstChild);
      }
      if (li !== children[children.length - 1]) {
        fragment.appendChild(document.createElement("br"));
      }
    });

    foundList.parentNode.replaceChild(fragment, foundList);

    nextTick(() => {
      if (lastInserted) {
        const newRange = document.createRange();
        newRange.setStartAfter(lastInserted);
        newRange.collapse(true);
        selection.removeAllRanges();
        selection.addRange(newRange);
      }

      recreateList(type);

      nextTick(() => {
        const newList = editor.value.querySelector(
          type === "unordered" ? "ul" : "ol"
        );
        if (newList?.lastElementChild) {
          const li = newList.lastElementChild;
          const range = document.createRange();
          range.selectNodeContents(li);
          range.collapse(false);
          selection.removeAllRanges();
          selection.addRange(range);
        }
        updateContent();
      });
    });
  } else {
    recreateList(type);

    nextTick(() => {
      const newList = editor.value.querySelector(
        type === "unordered" ? "ul" : "ol"
      );
      if (newList?.lastElementChild) {
        const li = newList.lastElementChild;
        const range = document.createRange();
        range.selectNodeContents(li);
        range.collapse(false);
        selection.removeAllRanges();
        selection.addRange(range);
      }
    });
  }
};

function applyCustomList(type = "ul") {
  const sel = window.getSelection();
  if (!sel.rangeCount) return;

  const range = sel.getRangeAt(0);
  const selectedText = range.toString();

  // Split lines
  const lines = selectedText
    .split(/\r?\n/)
    .filter((line) => line.trim() !== "");

  if (lines.length === 0) return;

  const list = document.createElement(type);

  for (const line of lines) {
    const li = document.createElement("li");
    li.textContent = line;
    list.appendChild(li);
  }

  // Replace selection with the new list
  range.deleteContents();
  range.insertNode(list);

  // Collapse selection after insert
  sel.removeAllRanges();
  const newRange = document.createRange();
  newRange.selectNodeContents(list);
  newRange.collapse(false);
  sel.addRange(newRange);
}

function triggerImageUpload() {
  fileInput.value.click();
}

function handleImageUpload(event) {
  const file = event.target.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = (e) => {
    insertImageDiv(e.target.result);
  };
  reader.readAsDataURL(file);
  event.target.value = "";
}

function insertImageDiv(imageSrc) {
  const container = editor.value;
  if (!container) return;

  const imgDiv = document.createElement("div");
  imgDiv.style.backgroundImage = `url("${imageSrc}")`;
  imgDiv.style.width = "300px";
  imgDiv.style.height = "200px";
  imgDiv.style.backgroundSize = "cover";
  imgDiv.style.backgroundPosition = "center";
  imgDiv.style.position = "absolute";
  imgDiv.style.border = "2px solid #ccc";
  imgDiv.style.zIndex = 1000;
  imgDiv.setAttribute("data-x", 0);
  imgDiv.setAttribute("data-y", 0);

  imgDiv.addEventListener("click", (e) => e.stopPropagation());

  container.appendChild(imgDiv);

  nextTick(() => {
    makeDraggable(imgDiv, container);
    makeResizable(imgDiv, container);
  });
}

function makeDraggable(el, container) {
  interact(el).draggable({
    inertia: true,
    modifiers: [
      interact.modifiers.restrictRect({
        restriction: container,
        endOnly: true,
      }),
    ],
    listeners: {
      move(event) {
        let x = (parseFloat(el.getAttribute("data-x")) || 0) + event.dx;
        let y = (parseFloat(el.getAttribute("data-y")) || 0) + event.dy;

        const maxX = container.clientWidth - el.offsetWidth;
        const maxY = container.clientHeight - el.offsetHeight;

        x = Math.max(0, Math.min(x, maxX));
        y = Math.max(0, Math.min(y, maxY));

        el.style.transform = `translate(${x}px, ${y}px)`;
        el.setAttribute("data-x", x);
        el.setAttribute("data-y", y);
      },
    },
  });
}

function makeResizable(el, container) {
  interact(el).resizable({
    edges: { top: true, left: true, bottom: true, right: true },
    modifiers: [
      interact.modifiers.restrictSize({
        min: { width: 100, height: 100 },
        max: {
          width: container.clientWidth,
          height: container.clientHeight,
        },
      }),
      interact.modifiers.restrictEdges({
        outer: container,
      }),
    ],
    listeners: {
      move(event) {
        const x =
          (parseFloat(el.getAttribute("data-x")) || 0) + event.deltaRect.left;
        const y =
          (parseFloat(el.getAttribute("data-y")) || 0) + event.deltaRect.top;

        const newWidth = event.rect.width;
        const newHeight = event.rect.height;

        // Prevent overflow from container
        const maxWidth = container.clientWidth - x;
        const maxHeight = container.clientHeight - y;

        el.style.width = `${Math.min(newWidth, maxWidth)}px`;
        el.style.height = `${Math.min(newHeight, maxHeight)}px`;
        el.style.transform = `translate(${x}px, ${y}px)`;
        el.setAttribute("data-x", x);
        el.setAttribute("data-y", y);
      },
    },
  });
}
function insertStyledButton() {
  const target = editor.value;
  if (!target) return;

  const button = document.createElement("button");
  button.innerText = "Click Me";
  button.classList.add("draggable-button");

  Object.assign(button.style, {
    position: "absolute",
    padding: "6px 10px",
    backgroundColor: "#f0f0f0",
    border: "1px solid #ccc",
    cursor: "move",
    userSelect: "text",
    zIndex: 1000,
    borderRadius: "4px",
  });

  button.setAttribute("data-draggable", "true");
  button.setAttribute("contenteditable", "false");
  button.style.left = "0px";
  button.style.top = "0px";
  button.setAttribute("data-x", 0);
  button.setAttribute("data-y", 0);

  // Show the popup on button click
  button.addEventListener("click", (e) => {
    e.stopPropagation();
    // Access the exposed showPopup method and call it
    popup.value.showPopup(button);
  });

  // Edit mode on double-click
  button.addEventListener("dblclick", (e) => {
    e.stopPropagation();
    button.setAttribute("contenteditable", "true");
    button.setAttribute("data-draggable", "false");
    button.focus();
    button.style.cursor = "text";
  });

  // Exit edit mode and re-enable drag
  button.addEventListener("blur", () => {
    button.setAttribute("contenteditable", "false");
    button.setAttribute("data-draggable", "true");
    button.style.cursor = "move";
    makeButtonDraggable(button, target);
  });

  button.addEventListener("keydown", (e) => {
    if (e.key === "Enter") {
      e.preventDefault();
      button.blur();
    }
  });

  target.appendChild(button);

  nextTick(() => {
    makeButtonDraggable(button, target);
  });
}

function makeButtonDraggable(button, container) {
  interact(button).draggable({
    inertia: true,
    modifiers: [
      interact.modifiers.restrictRect({
        restriction: container,
        endOnly: true,
      }),
    ],
    listeners: {
      move(event) {
        if (event.target.getAttribute("data-draggable") !== "true") return;

        let x = (parseFloat(button.getAttribute("data-x")) || 0) + event.dx;
        let y = (parseFloat(button.getAttribute("data-y")) || 0) + event.dy;

        const maxX = container.clientWidth - button.offsetWidth;
        const maxY = container.clientHeight - button.offsetHeight;

        x = Math.max(0, Math.min(x, maxX));
        y = Math.max(0, Math.min(y, maxY));

        button.style.transform = `translate(${x}px, ${y}px)`;
        button.setAttribute("data-x", x);
        button.setAttribute("data-y", y);
      },
    },
  });
}

function onEditorInput() {
  updateContent();
}

function insertGrid() {
  const target = editor.value;
  if (!target) return;

  const gridWrapper = document.createElement("div");
  gridWrapper.classList.add("grid-wrapper");
  gridWrapper.setAttribute("data-x", 0);
  gridWrapper.setAttribute("data-y", 0);
  gridWrapper.style.transform = `translate(0px, 0px)`;

  Object.assign(gridWrapper.style, {
    position: "absolute",
    top: "0",
    left: "0",
    padding: "10px",
    backgroundColor: "#f8f8f8",
    border: "2px dashed #aaa",
    display: "flex",
    flexDirection: "row",
    gap: "10px",
    zIndex: 10,
    overflow: "hidden", // Prevents overflow when resizing grid
  });

  // Make the entire grid resizable
  interact(gridWrapper).resizable({
    edges: { top: true, left: true, bottom: true, right: true },
    modifiers: [
      interact.modifiers.restrictEdges({
        outer: target,
      }),
    ],
    onmove(event) {
      const { x, y } = gridWrapper.dataset;
      gridWrapper.style.width = `${event.rect.width}px`;
      gridWrapper.style.height = `${event.rect.height}px`;
      gridWrapper.style.transform = `translate(${x}px, ${y}px)`;
    },
  });

  // Add default 2 columns
  for (let i = 0; i < 2; i++) {
    addColumn(gridWrapper);
  }

  // Add 'Add Column' Button
  const addBtn = document.createElement("button");
  addBtn.innerText = "+ Column";
  Object.assign(addBtn.style, {
    marginTop: "10px",
    cursor: "pointer",
  });

  addBtn.addEventListener("click", () => {
    addColumn(gridWrapper);
  });

  const container = document.createElement("div");
  container.appendChild(gridWrapper);
  container.appendChild(addBtn);
  target.appendChild(container);

  nextTick(() => {
    makeDraggableGrid(gridWrapper, target);
    Array.from(gridWrapper.children).forEach(makeResizableGrid); // Add resizable functionality to columns
  });
}

function addColumn(wrapper) {
  const cell = document.createElement("div");
  cell.contentEditable = "true";
  cell.innerText = "Editable cell";
  Object.assign(cell.style, {
    backgroundColor: "#fff",
    border: "1px solid #ccc",
    padding: "10px",
    minWidth: "100px",
    resize: "horizontal",
    overflow: "auto",
    position: "relative",
    // Remove flex to ensure resizing works
    width: "100px", // Set a fixed initial width for resizable functionality
    boxSizing: "border-box", // Prevent the box from overflowing its size
  });

  // Make the cell resizable
  makeResizableGrid(cell);

  wrapper.appendChild(cell);
}

function makeResizableGrid(element) {
  interact(element).resizable({
    edges: { left: true, right: true },
    modifiers: [
      interact.modifiers.restrictSize({
        min: { width: 50 }, // Minimum width for the column
      }),
    ],
    onmove(event) {
      const { width } = event.rect;
      element.style.width = `${width}px`;
    },
  });
}

function makeDraggableGrid(element, container) {
  interact(element).draggable({
    inertia: true,
    modifiers: [
      interact.modifiers.restrictRect({
        restriction: container,
        endOnly: true,
      }),
    ],
    listeners: {
      move(event) {
        let x = (parseFloat(element.getAttribute("data-x")) || 0) + event.dx;
        let y = (parseFloat(element.getAttribute("data-y")) || 0) + event.dy;

        const maxX = container.clientWidth - element.offsetWidth;
        const maxY = container.clientHeight - element.offsetHeight;

        x = Math.max(0, Math.min(x, maxX));
        y = Math.max(0, Math.min(y, maxY));

        element.style.transform = `translate(${x}px, ${y}px)`;
        element.setAttribute("data-x", x);
        element.setAttribute("data-y", y);
      },
    },
  });
}

function makeLink() {
  const url = prompt("Enter URL:");
  if (!url) return;

  const selection = window.getSelection();
  if (!selection.rangeCount) return;

  const range = selection.getRangeAt(0);
  const link = document.createElement("a");
  link.href = url;
  link.textContent = range.toString();
  link.target = "_blank";

  range.deleteContents();
  range.insertNode(link);
}
</script>

<style scoped>
.editor {
  white-space: nowrap;
}

.editor ul,
.editor ol {
  padding-left: 20px;
}

.editor-container {
  padding: 20px;
}

.editor {
  border: 1px solid #ccc;
  min-height: 200px;
  position: relative;
  padding: 10px;
}

.draggable-button {
  position: absolute;
}
.editor-container {
  padding: 20px;
}

.editor {
  border: 1px solid #ccc;
  min-height: 300px;
  position: relative;
  padding: 10px;
  background: #fff;
  overflow: hidden;
}

.tools {
  margin-top: 10px;
}
</style>
