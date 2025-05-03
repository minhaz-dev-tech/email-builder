<template>
  <div v-if="visible" class="modal-overlay" @click.self="closePopup">
    <div class="modal-content">
      <h3>Edit Button Styles</h3>
      <div>
        <label>
        Button Text:
          <input type="text" v-model="buttonStyle.text" />
        </label>
      </div>
      <div>
        <label>
          Background Color:
          <input type="color" v-model="buttonStyle.backgroundColor" />
        </label>
      </div>
      <div>
  <label>
    Button Link (URL):
    <input
      type="text"
      v-model="buttonStyle.link"
      placeholder="e.g. https://example.com"
    />
  </label>
     </div>

      <div>
        <label>
          Text Color:
          <input type="color" v-model="buttonStyle.color" />
        </label>
      </div>
      <div>
        <label>
          Padding:
          <input
            type="text"
            v-model="buttonStyle.padding"
            placeholder="e.g. 10px 20px"
          />
        </label>
      </div>
      <div>
  <label>
    Border Radius:
    <input
      type="text"
      v-model="buttonStyle.borderRadius"
      placeholder="e.g. 4px or 50%"
    />
  </label>
</div>

      <div>
        <button @click="applyStyle">Apply</button>
        <button @click="closePopup">Close</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const visible = ref(false);
const buttonStyle = ref({
  backgroundColor: "#f0f0f0",
  color: "#000000",
  padding: "6px 10px",
  borderRadius: "4px",
  text: "Click me",
  link: ""
});

const selectedButton = ref(null);

const showPopup = (button) => {
  visible.value = true;
  selectedButton.value = button;
  buttonStyle.value.backgroundColor = button.style.backgroundColor;
  buttonStyle.value.color = button.style.color;
  buttonStyle.value.padding = button.style.padding;
  buttonStyle.value.borderRadius = button.style.borderRadius;
  buttonStyle.value.text = button.textContent;
  buttonStyle.value.link = selectedButton.value.getAttribute("data-link") || "";

};


const closePopup = () => {
  visible.value = false;
};

const applyStyle = () => {
  if (!selectedButton.value) return;
  Object.assign(selectedButton.value.style, {
  backgroundColor: buttonStyle.value.backgroundColor,
  color: buttonStyle.value.color,
  padding: buttonStyle.value.padding,
  borderRadius: buttonStyle.value.borderRadius,
});
selectedButton.value.textContent = buttonStyle.value.text;

selectedButton.value.textContent = buttonStyle.value.text;
selectedButton.value.setAttribute("data-link", buttonStyle.value.link || "");
selectedButton.value.onclick = () => {
  if (buttonStyle.value.link) {
    window.open(buttonStyle.value.link, "_blank");
  }
};

  closePopup(); // Close the popup after applying styles
};

defineExpose({ showPopup }); // Expose the showPopup method
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  width: 300px;
}

.modal-content h3 {
  margin-top: 0;
}

.modal-content div {
  margin-bottom: 10px;
}

.modal-content label {
  display: block;
  margin-bottom: 5px;
}

.modal-content button {
  margin-top: 10px;
}
</style>
