## dr_interception
AMDのCPUの場合
仮想マシンのゲストがデバッグレジスタにアクセスしたときに
この関数が実行される

書き込み、読み込みした時
ハードウェアブレークポイントをセットした時

```
// linux\arch\x86\kvm\svm\svm.c
static int dr_interception(struct kvm_vcpu *vcpu)
{
	struct vcpu_svm *svm = to_svm(vcpu);
	int reg, dr;
	unsigned long val;
	int err = 0;

	/*
	 * SEV-ES intercepts DR7 only to disable guest debugging and the guest issues a VMGEXIT
	 * for DR7 write only. KVM cannot change DR7 (always swapped as type 'A') so return early.
	 */
	if (sev_es_guest(vcpu->kvm))
		return 1;

	if (vcpu->guest_debug == 0) {
		/*
		 * No more DR vmexits; force a reload of the debug registers
		 * and reenter on this instruction.  The next vmexit will
		 * retrieve the full state of the debug registers.
		 */
		clr_dr_intercepts(svm);
		vcpu->arch.switch_db_regs |= KVM_DEBUGREG_WONT_EXIT;
		return 1;
	}

	if (!boot_cpu_has(X86_FEATURE_DECODEASSISTS))
		return emulate_on_interception(vcpu);

	reg = svm->vmcb->control.exit_info_1 & SVM_EXITINFO_REG_MASK;
	dr = svm->vmcb->control.exit_code - SVM_EXIT_READ_DR0;
	if (dr >= 16) { /* mov to DRn  */
		dr -= 16;
		val = kvm_register_read(vcpu, reg);
		err = kvm_set_dr(vcpu, dr, val);
	} else {
		kvm_get_dr(vcpu, dr, &val);
		kvm_register_write(vcpu, reg, val);
	}

	return kvm_complete_insn_gp(vcpu, err);
}
```