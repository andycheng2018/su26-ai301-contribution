# su26-ai301-contribution

# Contribution 1: Instance uniform is stuck using color even after removing source_color hint

**Contribution Number:** 1  
**Student:** Andy Cheng  
**Issue:** https://github.com/godotengine/godot/issues/119973  
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose this issue because I use Godot myself to make games in my free time and wanted to contribute to an open-source project that I care about. This bug is related to Godot's shader behavior, which is something I'd like to learn more about by attempting to fix this bug. This issue felt like it was challenging but also realistic. I was able to replicate the bug myself and I hope to learn more about how Godot stores shader parameters and how the editor updates inspector values.

---

## Understanding the Issue

### Problem Description

[In your own words, what's broken or missing?]

### Expected Behavior

[What should happen?]

### Current Behavior

[What actually happens?]

### Affected Components

[Which parts of the codebase are involved?]

---

## Reproduction Process

### Environment Setup

- Godot 4.7 beta4
- Downloaded the editor from the official Godot website
- No special dependencies needed

### Steps to Reproduce

1. Open Godot 4.7 beta4 and create a new 2D project
2. Add a ColorRect node to the scene
3. Create a new ShaderMaterial on the ColorRect and paste in the following shader:

shader_type canvas_item;

instance uniform vec3 color : source_color;

void fragment() {
    COLOR.rgb = color;
}
   
5. In the Inspector, expand the shader parameters and assign a color value to the color instance uniform — use something like #3a504d so the sRGB ↔ Linear conversion is visibly affected
Edit the shader and remove the source_color hint, so the line becomes: instance uniform vec3 color;
6. Save the shader — observe that the uniform editor in the Inspector still shows a Color picker instead of switching to a Vector3 input
7. Try to revert or re-enter a value — the Color-converted value persists; the only workarounds are pasting a raw Vector3/4 value or manually editing the .tscn file to remove the stored value

### Reproduction Evidence

- **Commit showing reproduction:** https://github.com/andycheng2018/godot
- **Screenshots/logs:** See video in here: https://github.com/godotengine/godot/issues/119973 
- **My findings:** The root cause appears to be that Godot caches the declared type hint used when the Color value was first assigned, and stores it in the scene file (.tscn). When the shader is reloaded without source_color, the editor reads the stored Color variant from the scene instead of re-inferring the type from the updated shader, so it keeps rendering the Color widget. The scene data overrides the shader's current declaration.
---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

- Locate the uniform type resolution code — find where the editor inspector determines which editing widget to show for shader instance uniforms (likely in scene/resources/shader.cpp, editor/plugins/shader_editor_plugin.cpp, or the material property inspector)
- Identify where the cached/stored variant type wins over the shader's declared type — the bug is that on shader reload, the stored scene value's type (Color) is being used to pick the widget instead of the newly parsed shader parameter type (vec3 without hint)
- Force widget refresh on shader recompile — when a shader is modified and reloaded, instance uniform editor widgets should be invalidated and re-created based on the current shader parameter declarations, not the type of the previously stored value
- Handle type mismatch between stored value and shader declaration — if the stored variant type no longer matches the shader uniform's type/hint, either discard the stored value, convert it, or warn the user, rather than silently using the wrong widget
- Test: confirm that after removing source_color, saving, and reopening the Inspector, the uniform shows a Vector3 input and that reverting the value works correctly

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
