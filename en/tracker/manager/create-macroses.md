# Macros

In {{ tracker-name }}, macros are scripted algorithms that can be executed on the issue page. You can use macros to automate repeating actions. Macros let you change issue fields, create automated comments in just one click.

## Creating a macro {#section_inq_5b1_x2b}

{% note warning %}

By default, [only the queue owner](queue-access.md) can configure a queue.

{% endnote %}

Each {{ tracker-name }} queue has its own set of macros. To create a new queue macro, do the following:

1. Open the [queue page](../user/queue.md).

1. To the left of the queue name, select ![](../../_assets/tracker/icon-settings.png) → **Administration**.

1. On the left-hand panel, select **Automation** → **Macros** and click **Create macro**.

1. Set up your macro parameters:
    - **Macro name**.
    - **Message**: Message body (comment) created when executing a macro. If you don't want your macro to create any messages, leave this field empty.
You can insert field values into your comments. To do this, click **Add variable** and select one or more values. The **Message** field will show a sequence like `not_var{{issue.fieldKey}}`.
    - **Actions**: Choose issue fields a macro should change and specify their values after the change.

1. Click **Create macro**.

## Editing and deleting macros {#section_swl_sdb_x2b}

{% note warning %}

By default, [only the queue owner](queue-access.md) can configure a queue.

{% endnote %}

1. Open the [queue page](../user/queue.md).

1. To the left of the queue name, select ![](../../_assets/tracker/icon-settings.png) → **Administration**.

1. On the left-hand panel, select **Automation** → **Macros** and hover over the macro you need.

1. To edit the macro, click ![](../../_assets/tracker/icon-edit.png).
To delete a macro, click ![](../../_assets/tracker/icon-delete.png).

## Running a macro {#section_ekq_22b_x2b}

Macros let you change issue fields, create automated comments. Any user with access rights to issue editing can execute macros.

To run a macro:

1. Open the issue page.

1. Go to the comment field.

1. Select a macro from the **Macros** drop-down list.
You can select multiple macros at the same time. If multiple macros change the same field, only the last executed change will be applied.


1. To execute the macro, click **Submit**.

## Example of a macro {#macro_example}


Let's say a member of the first line of support wants to transfer a user request in {{ tracker-name }} to the second line of support. We'll set up a macro that does exactly that:

1. Choose the queue where you want to create the macro for switching the request to a different support line and open its settings.

1. In the **Macros** section, click [**Create macro**](#section_inq_5b1_x2b).

1. Set the macro name.

1. Write the text to add to the issue comment. You can add issue fields by clicking **Add variable**.

1. If you want your macro to assign an issue to a specific employee in the second support line, find the **Actions** section, choose **System** → **Assignee**, and specify the employee's name.

1. If you want your macro to change the issue status, go to **Actions**, choose **System** → **Status**, and set it to **Support line 2**.
If the desired status is not in the list of values, [set up a workflow](add-workflow.md).

   ![](../../_assets/tracker/macro-example-line2.png)

1. Save your macro.

To run a macro you created:

1. Open any issue from the queue you made the macro in.

1. Click **Macros** in the comment box and choose a name for your macro.

1. Your text is automatically added to the comment. If necessary, edit it.

1. Click **Submit**. The issue will be transferred to the second support line.

