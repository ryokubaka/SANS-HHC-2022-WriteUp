# AWS CLI Intro

### 1:

![](../../../resources/screenshots/awscliintro-1.png)

- So, we type `aws help`

---

### 2:

![](../../../resources/screenshots/awscliintro-2.png)

- It then asks us to configure the default `aws cli` credentials with the provided access and secret key. We run `aws configure` and enter in the values. I did re-run it to ensure I put `json` as my default output format.

![](../../../resources/screenshots/awscliintro-2-ans.png)

---

### 3:

![](../../../resources/screenshots/awscliintro-3.png)

- It asks us to get our caller identity, so we run `aws sts get-caller-identity`:

![](../../../resources/screenshots/awscliintro-3-sol.png)

And that was it! Back to the [cloud ring room](../README.md).