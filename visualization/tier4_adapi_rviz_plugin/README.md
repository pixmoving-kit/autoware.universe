# tier4_adapi_rviz_plugin

此功能包包含用于测试 AD API 的工具。对于常规 AD API 使用，建议使用 [tier4_state_rviz_plugin](../tier4_state_rviz_plugin/README.md)。

## RoutePanel

使用此面板时，请将 2D Goal Pose Tool 的话题名称设置为 `/rviz/routing/pose`。
默认情况下，工具发布位姿后，面板会立即以该位姿为目标设置路线。
可以通过复选框启用或禁用 allow_goal_modification 选项。

点击途经点区域中的模式按钮，进入途经点模式。在此模式下，位姿会添加到途经点列表中。
点击 apply 按钮，使用保存的途经点设置路线（最后一个点为目标位置）。
使用 reset 按钮清空已保存的途经点。

## Material Design Icons

This project uses [Material Design Icons](https://developers.google.com/fonts/docs/material_symbols) by Google. These icons are used under the terms of the Apache License, Version 2.0.

Material Design Icons are a collection of symbols provided by Google that are used to enhance the user interface of applications, websites, and other digital products.

### License

The Material Design Icons are licensed under the Apache License, Version 2.0. You may obtain a copy of the License at:

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

### Acknowledgments

We would like to express our gratitude to Google for making these icons available to the community, helping developers and designers enhance the visual appeal and user experience of their projects.
