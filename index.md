/****************************************************************************************************************
		 Copyright Notice:			Children's Hospital of the King's Daughters Health System (CHKDHS)
*****************************************************************************************************************
 
		 Author:       				DeAnn Capanna
		 Date Written:  			08/10/2020
		 Source file name: 			chkd_csg_rpt_ed_prvnotetat.prg
		 Object name:   			chkd_csg_rpt_ed_prvnotetat
 
		 Program purpose:			CSG ED provider note TAT by encounter.
 
		 Special Notes:				None
 
;****************************************************************************************************************
 
                                             MODIFICATION CONTROL LOG
 
;****************************************************************************************************************
 
   Ver  Date            Engineer                Description
   ---  --------------- ----------------------- ----------------------------------------------------------------
   001  08/10/2020      DeAnn Capanna           Initial Release
   002  mm/dd/yyyy		engineer name			Notes on modification
   006  10/22/2020		DeAnn Capanna			Modifications per Michelle Green's email dated 9/15/2020
   009	11/19/2020		DeAnn Capanna			Modifications per meeting on 11/19/2020
   010	12/08/2020      DeAnn Capanna			Modifications per Michelle Green's email dated 12/7/2020
   012  01/12/2021		DeAnn Capanna			Add MD Median CSV output.
   025	02/15/2021		DeAnn Capanna			Added LEWIS NP, MICHAELA N	LEWISMN	   	   33113074.00 CSG Emergency Medicine
													  MURPHY PA-C, KELLEY M	MURPHYKM	   33113092.00 CSG Emergency Medicine
;***************************************************************************************************************/
drop program chkd_csg_rpt_ed_prvnoteta4:group1 go
create program chkd_csg_rpt_ed_prvnoteta4:group1
/*===============================================================================================================
 
                                          		     PROMPTS
 
===============================================================================================================*/
prompt
	"Output to File/Printer/MINE" = "MINE"
	, "Performed Begin Date:" = "CURDATE"
	, "Performed End Date:" = "CURDATE"
	, "Rpt2" = 0
	, "Choose Report:" = 2
	, "Ops" = 0
 
with OUTDEV, pSTART_DATE, pEND_DATE, Rpt2, Rpt, Ops
/*===============================================================================================================
 
                                          		DECLARED SUBROUTINES
 
===============================================================================================================*/
/*===============================================================================================================
 
                                          		DEFINED RECORDSETS
 
===============================================================================================================*/
call echo(concat("START:    ", format(curtime3, "HH:MM:SS:CC;;M")))
 
free record temp
record  temp  (
 1 begin_date							= vc
 1 end_date								= vc
 1 date_range_disp						= vc
 1 enc_cnt  							= i4
 1 rec[*]
 	2 pt_encntr_id						= f8
 	2 pt_person_id						= f8
 	2 tid								= f8
 	2 pt_mrn  							= vc
 	2 pt_fin							= vc
 	2 pt_name							= vc
 	2 pt_type							= vc
 	2 pt_med_srvc						= vc
 	2 pt_med_service_cd					= f8
 	2 esi								= i2
 	2 encntr_reason_for_visit			= vc
 	2 note_to_biller					= vc
 	2 pt_facility						= vc
 	2 pt_unit							= vc
 	2 pt_room							= vc
 	2 pt_bed							= vc
 	2 pt_admit							= vc
 	2 checkin_dt_tm						= vc
	2 checkin_dt_tm_dq8					= dq8
	2 depart_dt_tm						= vc
	2 depart_dt_tm_dq8					= dq8
	2 los								= f8
	2 disposition						= vc
 	2 provider_id						= f8
 	2 provider 							= vc
 	2 provider_position					= vc
 	2 prov_role							= vc
 	2 department						= vc
 	2 event_id							= f8
 	2 parent_event_id					= f8
	2 event_cd							= f8
	2 parent_event_id					= f8
	2 event_class_disp					= vc
	2 result_status_disp				= vc
	2 rpt 								= vc
	2 pt_dos							= vc
	2 pt_dos_dq8						= dq8
	2 note_provider						= vc
	2 note_provider_id					= f8
	2 encntr_prsnl_r_disp				= vc
	2 encntr_prsnl_r_cd					= f8
	2 signed 							= vc
 	2 signed_dq8						= dq8
 	2 signed_diff						= vc
 	2 signed_diff_f8					= f8
 	2 second_signed_dq8					= dq8
 	2 second_signed_diff				= vc
 	2 second_signed_diff_f8				= f8
 	2 num_not_signed					= f8
 	2 num_signed						= f8
 	2 num_note							= i4
 	2 Trans_from_Author_to_Attending_dq8= dq8
 	2 Transfer_from_Author_to_Attending = vc
 	2 trans_attending					= vc
 	2 trans_attending_id				= f8
 	2 second_signed_dq8					= dq8
 	2 second_signed						= vc
 
 	2 phy_assigned 						= vc
	2 phy_assigned_dt_tm 				= vc
	2 arrival_to_1stdoc					= i4
	2 att_assigned 						= vc
	2 att_assigned_dt_tm				= vc
 
	2 provs[*]
		2 resident						= vc
		2 res_position					= vc
		2 res_performed_dt_tm 			= vc
		2 res_sign_dt_tm				= vc
		2 res_signed_dt_tm				= vc
 
	2 endo_assigned						= vc
 
	2 result_status						= vc
	2 result_title						= vc
 
)
 
;Record structure for Median calculations
free record temp2
record temp2(
	1 begin_date						= vc
	1 end_date							= vc
	1 date_range_disp					= vc
	1 dept_cnt							= i4
	1 dept[*]
		2 department					= vc
		2 num_lag_days					= f8
		2 num_notes						= f8
		2 num_encntrs					= f8
		2 num_within_1_day 				= i4
		2 num_within_2_days 			= i4
		2 num_within_3_days 			= i4
		2 num_within_4_days 			= i4
		2 him_num_within_1_day 				= i4
		2 him_num_within_2_days 			= i4
		2 him_num_within_3_days 			= i4
		2 him_num_within_4_days 			= i4
		2 num_signed					= f8
 		2 num_not_signed				= f8
		2 av_lag_days					= f8
		2 median						= f8
		2 median_vc						= vc
		2 percent_within_1_day 			= f8
		2 percent_within_2_days 		= f8
		2 percent_within_3_days 		= f8
		2 percent_within_4_days 		= f8
 
		2 him_av_lag_days					= f8
		2 him_median						= f8
		2 him_median_vc						= vc
		2 him_percent_within_1_day 			= f8
		2 him_percent_within_2_days 		= f8
		2 him_percent_within_3_days 		= f8
		2 him_percent_within_4_days 		= f8
 
		2 percent_signed				= f8
		2 percent_not_signed			= f8
 
		2 percent_within_1_day_vc		= vc
		2 percent_within_2_days_vc 		= vc
		2 percent_within_3_days_vc		= vc
		2 percent_within_4_days_vc 		= vc
 
		2 him_percent_within_1_day_vc		= vc
		2 him_percent_within_2_days_vc 		= vc
		2 him_percent_within_3_days_vc		= vc
		2 him_percent_within_4_days_vc 		= vc
 
		2 percent_signed_vc				= vc
		2 percent_not_signed_vc			= vc
		2 prov_cnt						= i4
		2 prov[*]
			3 prov_id					= f8
			3 provider					= vc
 
			3 num_encntrs				= f8
 
			3 num_notes					= f8
			3 num_signed				= f8
			3 num_not_signed			= f8
 
			3 num_within_1_day 			= i4
			3 num_within_2_days 		= i4
			3 num_within_3_days 		= i4
			3 num_within_4_days 		= i4
 
			3 him_num_within_1_day 			= i4
			3 him_num_within_2_days 		= i4
			3 him_num_within_3_days 		= i4
			3 him_num_within_4_days 		= i4
 
			3 percent_within_1_day 		= f8
			3 percent_within_2_days 	= f8
			3 percent_within_3_days 	= f8
			3 percent_within_4_days 	= f8
			3 percent_within_1_day_vc	= vc
			3 percent_within_2_days_vc 	= vc
			3 percent_within_3_days_vc	= vc
			3 percent_within_4_days_vc 	= vc
 
			3 him_percent_within_1_day 		= f8
			3 him_percent_within_2_days 	= f8
			3 him_percent_within_3_days 	= f8
			3 him_percent_within_4_days 	= f8
			3 him_percent_within_1_day_vc	= vc
			3 him_percent_within_2_days_vc 	= vc
			3 him_percent_within_3_days_vc	= vc
			3 him_percent_within_4_days_vc 	= vc
 
			3 percent_not_signed		= f8
			3 percent_not_signed_vc		= vc
 
			3 median					= f8
 			3 median_vc					= vc
 
 			3 him_median					= f8
 			3 him_median_vc					= vc
				;*************DELETE*********
			3 num_lag_days				= f8
			3 av_lag_days				= f8
			3 percent_signed			= f8
			3 percent_signed_vc			= vc
 
)
/*===============================================================================================================
 
                                             PROGRAMMER CONSTANTS
 
===============================================================================================================*/
declare TESTENV_IND			= i1 with protect, constant(evaluate(currdbname, "PROD", 0, 1))
 
declare FIN_319_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 319, "FIN NBR"))
declare MRN_4_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 4, "MRN"))
 
;Result Status Codes
declare ANT_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "ANTICIPATED"))
declare AUTH_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "AUTH"))
declare INPROG_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "IN PROGRESS"))
declare ALT_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "ALTERED"))
declare MOD_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "MODIFIED"))
declare TRANS_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "TRANSCRIBED"))
declare UNAUTH_8_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 8, "UNAUTH"))
 
;checkout_disposition_cd
declare LEFTWOBEINSEEN_19_CV= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 19, "04LEFTWITHOUTBEINGSEEN"))
 
declare	PERFORM_21_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 21, "PERFORM"))
declare AUTHOR_21_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 21, "AUTHOR"))
declare SIGN_21_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 21, "SIGN"))
declare VERIFY_21_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 21, "VERIFY"))
declare TRANSCRIBE_21_CV	= f8 with protect, constant(uar_get_code_by("MEANING", 21, "TRANSCRIBE"))
 
;active_status_cd
declare ACTIVE_48_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 48, "ACTIVE"))
 
;Event Class Codes
declare MDOC_53_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 53, "MDOC"))
declare DOC_53_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 53, "DOC"))
declare TXT_53_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 53, "TXT"))
declare GRPDOC_53_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 53, "GRPDOC"))
declare CLINDOC_53_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 53, "CLINDOC"))
declare DOCUMENT_53_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 53, "DOCUMENT"))
declare SCDOCUMENT_53_CV	= f8 with protect, constant(uar_get_code_by("MEANING", 53, "SCDOCUMENT"))
 
;admit_type_Cd
declare EMERGENCY_71_CV		= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 71, "EMERGENCY"))
 
;NOTES USED IN ED
declare KDEDPOWERNOTE_72_CV = f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 72, "KDEDPOWERNOTE"))
declare KDEDMDDICTATION_72_CV= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 72, "KDEDMDDICTATION"))
declare KDEDMDDICTATIO_72_CV		= f8 with protect, constant(uar_get_code_by("DESCRIPTION", 72, "KD ER-E.."))
declare KDEDMDWRITRCRD_72_CV		= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 72, "KDEDMDWRITTENRECORD"))
 
;position_cd from prsnl table
declare KDRESIDENT_88_CV			= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 88, "KDRESIDENT"))
 
;action_status_cd
declare COMPLETED_103_CV 			= f8 with protect, constant(uar_get_code_by("MEANING", 103, "COMPLETED"))
declare REQUESTED_103_CV			= f8 with protect, constant(uar_get_code_by("MEANING", 103, "REQUESTED"))
 
;encntr_prsnl_r_cd code set 333
declare RESIDENT_333_CV				= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 333, "RESIDENT"))
declare ATTENDINGROUNDS_333_CV		= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 333, "ATTENDINGROUNDS"))
declare ATTENDINGPHYSICIAN_333_CV	= f8 with protect, constant(1119.00)
declare ALLIEDHEALTH_333_CV			= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 333, "ALLIEDHEALTH"))
declare ASSIGNED_333_CV				= f8 with protect, constant(uar_get_code_by("MEANING", 333, "ASSIGNED"))
declare ASSIGNEDPAT_333_CV 			= f8 with protect, constant(uar_get_code_by("MEANING", 333, "ASSIGNED PAT"))
;tracking_group_cd code set 16370
declare EDTRKGRP_16370_CV 	= f8 with protect, constant(uar_get_code_by("DISPLAYKEY", 16370, "CHKDEDTRACKINGGROUP"))
 
;entry_mode_cd
declare ESI_29520_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 29520, "ESI"))
declare DYNDOC_29520_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 29520, "DYNDOC"))
 
;order_status_cd
declare ORDERED_6004_CV		= f8 with protect, constant(uar_get_code_by("MEANING", 6004, "ORDERED"))
declare TRANSCANCEL_6004_CV = f8 with protect, constant(uar_get_code_by("MEANING", 6004, "TRANS/CANCEL"))
 
;acuity_level_id
declare ACLVL1_ID			= f8 with protect, constant(12108759.00)
declare ACLVL2_ID			= f8 with protect, constant(12109522.00)
declare ACLVL3_ID			= f8 with protect, constant(12109526.00)
declare ACLVL4_ID			= f8 with protect, constant(12109530.00)
declare ACLVL5_ID			= f8 with protect, constant(12109534.00)
declare ACLVLU_ID			= f8 with protect, constant(0.00)
 
;CSG EMERGENCY MEDICINE Providers
declare GUINSTE_ID			= f8 with protect, constant(917592.00)		;GUINS MD, THERESA E	GUINSTE
declare RAMIREDE_ID			= f8 with protect, constant(894708.00)		;RAMIREZ MD, DANA E	RAMIREDE
declare	POIRIEMP_ID			= f8 with protect, constant(917617.00)		;POIRIER MD, MICHAEL P	POIRIEMP
declare QURESHFA_ID			= f8 with protect, constant(914452.00)		;QURESHI MD, FAIQA A	QURESHFA
declare	SARTORSC_ID			= f8 with protect, constant(894447.00)		;SARTORI MD, SUZANNE	SARTORSC
declare CLINGEJM_ID			= f8 with protect, constant(894693.00)		;CLINGENPEEL MD, JOEL M	CLINGEJM
declare MILLERJD_ID			= f8 with protect, constant(894612.00)		;MILLER MD, JILL D.	MILLERJD
declare PETRONKA_ID			= f8 with protect, constant(894616.00)		;PETRONIS MD, KELLI A.	PETRONKA
declare	WHITENJ_ID			= f8 with protect, constant(894619.00)		;WHITE MD, NICHOLAS	WHITENJ
declare SCHMIDJM_ID			= f8 with protect, constant(894672.00)		;SCHMIDT MD, JAMES	SCHMIDJM
declare GODAMBS_ID			= f8 with protect, constant(25819103.00)	;GODAMBE MD, SANDIP	GODAMBS
declare ARZUBIMK_ID			= f8 with protect, constant(26086808.00)	;ARZUBI-HUGHES MD, MICHELLE K.	ARZUBIMK
declare KAPOORR_ID			= f8 with protect, constant(28030793.00)	;KAPOOR MD, RUPA	KAPOORR
declare BURHOPJE_ID			= f8 with protect, constant(26069479.00)	;BURHOP DO, JAMES E	BURHOPJE
declare MULLANPC_ID			= f8 with protect, constant(29249048.00)	;MULLAN MD, PAUL C.	MULLANPC
declare SCHACHNM_ID			= f8 with protect, constant(1202951.00)		;SCHACHERER MD, NICOLE M.	SCHACHNM
declare EASONMK_ID			= f8 with protect, constant(2630474.00)		;EASON MD, MARGARET KELLY	EASONMK
declare LEADERAP_ID			= f8 with protect, constant(28508859.00)	;LEADER MD, ALEXANDRA P.	LEADERAP
declare WELLERMA_ID			= f8 with protect, constant(29194988.00)	;WELLER MD, MELANIE A	WELLERMA
declare HERBERKE_ID			= f8 with protect, constant(29194987.00)	;HERBERT DO, KRISTIN E.	HERBERKE
declare GALIOTJL_ID			= f8 with protect, constant(894769.00)		;GALIOTOS MD, JENNIFER LEPERE	GALIOTJL
declare VOKOUNKJ_ID			= f8 with protect, constant(2125993.00)		;VOKOUN MD, KELLY J.	VOKOUNKJ
declare GABRIENM_ID			= f8 with protect, constant(917460.00)		;GABRIEL MD, NOELLE M	GABRIENM
declare BLANCOOA_ID			= f8 with protect, constant(894476.00)		;BLANCO MD, OMAR A	BLANCOOA
declare HORNBUAC_ID			= f8 with protect, constant(894546.00)		;HORNBUCKLE MD, ANDREA	HORNBUAC
declare MILLERSF1_ID		= f8 with protect, constant(894617.00)		;MILLER III MD, STEPHEN F	MILLERSF1
declare SCHOCKKK_ID			= f8 with protect, constant(894663.00)		;SCHOCK MD, KIM K	SCHOCKKK
declare PHILIPV_ID			= f8 with protect, constant(30599094.00)	;PHILIP MD, VIPIN	PHILIPV
declare WEAVERBJ_ID			= f8 with protect, constant(30599106.00)	;WEAVER MD, BYRON J.	WEAVERBJ
declare ATAYO_ID			= f8 with protect, constant(25996389.00)	;ATAY MD, ORHAN	ATAYO
declare ASMUNDAS_ID			= f8 with protect, constant(31484544.00)	;ASMUNDSSON MD, ANNA S E	ASMUNDAS
;1881	MD	Carr, Daniel T.    32622595.00 --- does not have a username
declare FARMERPF_ID			= f8 with protect, constant(29819028.00)	;FARMER MD, PETER F.	FARMERPF
declare THOMASPM_ID			= f8 with protect, constant(29819052.00)	;THOMAS MD, PHILLIP M.	THOMASPM
declare WALLENAN_ID			= f8 with protect, constant(22493401.00)	;WALLEN DO, ANDRIA N	WALLENAN
declare WOODAE1_ID			= f8 with protect, constant(31360854.00)	;WOOD DO, ALLISON E	   	WOODAE1
declare PROCTODE_ID			= f8 with protect, constant(31354961.00)	;PROCTOR MD, DREXEL E	PROCTODE
declare WATERHOD_ID			= f8 with protect, constant(914304.00)		;WATERHOUSE PNP, DIANE	WATERHOD
declare CLEMENLM_ID			= f8 with protect, constant(28442838.00)	;CLEMENTS PNP, LYNDSEY M	CLEMENLM
declare HICKSKS_ID			= f8 with protect, constant(29297362.00)	;HICKS CPNP, KELLY S	HICKSKS
declare GREENJJ_ID			= f8 with protect, constant(24128286.00)	;GREEN PNP, JESSICA J	GREENJJ
declare MALDONLL_ID			= f8 with protect, constant(32212931.00)	;MALDONADO NP, LINDA L	MALDONLL
declare HENEBRMP_ID			= f8 with protect, constant(32414098.00)	;HENEBRY NP, MIRIAM P	HENEBRMP
;v3
declare GABRIEC1_ID			= f8 with protect, constant(1168247.00)		;GABRIEL MD, CANDICE A.	GABRIEC1
declare SMITHSE3_ID			= f8 with protect, constant(30579995.00)	;SMITH MD, SARA E.	SMITHSE3
declare KULANDG_ID			= f8 with protect, constant(30579626.00)	;KULANDAIVEL MD, GOWRY	KULANDG
;v15
declare LEWISMN_ID			= f8 with protect, constant(33113074.00)	;LEWIS NP, MICHAELA N	LEWISMN	   	    CSG Emergency Medicine
declare MURPHYKM_ID			= f8 with protect, constant(33113092.00)	;MURPHY PA-C, KELLEY M	MURPHYKM	    CSG Emergency Medicine
;v16
 
;;;;;TEST TOTAL ENCOUNTERS
/*select distinct
ti.encntr_id
from
	TRACKING_CHECKIN   TC
	, tracking_item ti
plan TC where tc.tracking_group_cd =    12108058.00
	and tc.checkout_dt_tm between cnvtdatetime(cnvtdate(02012021),0) and cnvtdatetime(cnvtdate(02282021),2359)
	and tc.active_ind = 1
	;and tc.checkout_dt_tm < cnvtdatetime(curdate,curtime)
	and tc.checkout_disposition_cd !=     3980299.00
	and tc.acuity_level_id in (12108759.00, 12109522.00, 12109526.00, 12109530.00, 12109534.00, 0.00)
join ti where ti.tracking_id = tc.tracking_id
	and ti.encntr_id > 0
order by ti.encntr_id
with nocounter
/*===============================================================================================================
 
                                           PROGRAMMER DEFINED VARIABLES
 
===============================================================================================================*/
declare num					= i4 with protect, noconstant(0)	;Variable for expand
declare idx					= i4 with protect, noconstant(0) 	;variable for locateval
declare num2				= i4 with protect, noconstant(0)	;Variable for expand
 
declare pos					= i4 with protect, noconstant(0)	;Variable for findstring
declare pos2				= i4 with protect, noconstant(0)	;Variable for findstring
declare pos3				= i4 with protect, noconstant(0)	;Variable for findstring
declare pos4				= i4 with protect, noconstant(0)	;Variable for findstring
declare tempstring			= vc with protect, noconstant("")	;Variable for findstring
 
declare deparment			= vc with protect, noconstant("")
declare deparment_list		= vc with protect, noconstant("")
 
declare cnt					= i4 with protect, noconstant(0)
declare par 				= c20
set lnum 					= 0
set num						= 4
set items					= 48
declare parsize				= i4
 
;Blob
declare OCFCOMP_120_CV		= f8 with protect, Constant(uar_get_code_by("MEANING", 120, "OCFCOMP"))
declare NOCOMP_120_CV 		= f8 with protect, Constant(uar_get_code_by("MEANING", 120, "NOCOMP"))
 
declare blobin             	= gc32768 with noconstant("")
declare blobout            	= gc32768 with noconstant("")
declare blobnortf          	= gc32768 with noconstant("")
 
declare blob_ret_len 	   	= w8 with noconstant(0)
 
declare i					= i4 with protect, noconstant(0)
 
declare start_date 			= dq8
declare end_date 			= dq8
 
declare date_range_disp		= vc with protect, noconstant("")
 
declare vProviders			= vc
 
declare check				= i4
/*===============================================================================================================
 
                                            GATHER PROMPT INFORMATION
 
===============================================================================================================*/
if (isnumeric($pSTART_DATE) > 0) ;input was in curdate/curtime format
	set start_date 		= cnvtdatetime($pSTART_DATE, 0)
else ;input was in dd-mmm-yyyy string format
	set start_date 		= cnvtdatetime(cnvtdate2($pSTART_DATE, "dd-mmm-yyyy"), 0)
endif
 
if (isnumeric($pEND_DATE) > 0) ;input was in curdate/curtime format
	set end_date 		= cnvtdatetime($pEND_DATE, 235959)
else ;input was in dd-mmm-yyyy string format
	set end_date 		= cnvtdatetime(cnvtdate2($pEND_DATE, "dd-mmm-yyyy"), 235959)
endif
 
set temp->begin_date 	= format(start_date, "mm/dd/yyyy;;d")
set temp->end_date		= format(end_date, "mm/dd/yyyy;;d")
 
if(datetimediff(cnvtdatetime(end_date), cnvtdatetime(start_date)) > 31)
	select into $OUTDEV
 		WARNING = "CHOOSE A DATE RANGE THAT IS <= 31 DAYS"
 	with NOCOUNTER, SEPARATOR = " ", FORMAT
 
 	go to exit_script
endif
 
set temp->date_range_disp= concat("From ", format(start_date, "dd-mmm-yyyy hh:mm;;d"),
							 	  " To ", format(end_date, "dd-mmm-yyyy hh:mm;;d"))
 
set temp2->date_range_disp= concat("From ", format(start_date, "dd-mmm-yyyy hh:mm;;d"),
							 	  " To ", format(end_date, "dd-mmm-yyyy hh:mm;;d"))
 
call echo(temp->date_range_disp)
/*===============================================================================================================
 
                                       GET CSG EMERGENCY ENCOUNTERS
 
===============================================================================================================*/
call echo(concat("GET CSG EMERGENCY ENCOUNTERS query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
select into "NL:"
from
	tracking_checkin tc
	, tracking_item ti
	, encounter e
	, person p
	, him_event_allocation hea
	, clinical_event ce
	, prsnl prov
	, encntr_alias fin
	, person_alias mrn
 
plan tc
	where tc.tracking_group_cd = EDTRKGRP_16370_CV
	  and tc.checkout_dt_tm between cnvtdatetime(start_date) and cnvtdatetime(end_date) ;DOS = checkout_dt_Tm
	  and tc.active_ind = 1
	  and tc.checkout_disposition_cd != LEFTWOBEINSEEN_19_CV
	  and tc.acuity_level_id in (ACLVL1_ID, ACLVL2_ID, ACLVL3_ID, ACLVL4_ID, ACLVL5_ID, ACLVLU_ID)
 
join ti
	where ti.tracking_id = tc.tracking_id
	  and ti.encntr_id > 0
 
join e
	where e.encntr_id = ti.encntr_id
	  and e.end_effective_dt_tm > cnvtdatetime(curdate, curtime3)
	  and e.active_ind = 1
 
join p
	where p.person_id = e.person_id
 
join hea
	where hea.encntr_id = ti.encntr_id
	  and hea.active_ind = 1
	  and hea.end_effective_dt_tm = cnvtdate(12312100)
 
join ce
	where ce.event_id = hea.event_id
	  and ce.event_cd in (KDEDPOWERNOTE_72_CV, KDEDMDDICTATIO_72_CV, KDEDMDWRITRCRD_72_CV)
	  and ce.event_end_dt_tm <= cnvtdatetime(curdate, curtime3)
	  and ce.valid_until_dt_tm = cnvtdate(12312100)
;	  and ce.result_status_cd in (AUTH_8_CV
;	  							, ALT_8_CV
;	  							, MOD_8_CV
;	  						    , TRANS_8_CV)
	  and cnvtupper(ce.event_title_text) != "DATE\TIME CORRECTION"
	  and cnvtupper(ce.event_tag) != "IN ERROR*"
	  and ce.view_level = 1
	  and ce.authentic_flag = 1
 
join prov
	where prov.person_id = hea.prsnl_id
	  and prov.physician_ind = 1
	  /*and prov.person_id in (GUINSTE_ID
							, RAMIREDE_ID
							, POIRIEMP_ID
							, QURESHFA_ID
							, SARTORSC_ID
							, CLINGEJM_ID
							, MILLERJD_ID
							, PETRONKA_ID
							, WHITENJ_ID
							, SCHMIDJM_ID
							, GODAMBS_ID
							, ARZUBIMK_ID
							, KAPOORR_ID
							, BURHOPJE_ID
							, MULLANPC_ID
							, SCHACHNM_ID
							, EASONMK_ID
							, LEADERAP_ID
							, WELLERMA_ID
							, HERBERKE_ID
							, GALIOTJL_ID
							, VOKOUNKJ_ID
							, GABRIENM_ID
							, BLANCOOA_ID
							, HORNBUAC_ID
							, MILLERSF1_ID
							, SCHOCKKK_ID
							, PHILIPV_ID
							, WEAVERBJ_ID
							, ATAYO_ID
							, ASMUNDAS_ID
							;1881	MD	Carr, Daniel T.    32622595.00 <<<--- does not have a username
							, FARMERPF_ID
							, THOMASPM_ID
							, WALLENAN_ID
							, WOODAE1_ID
							, PROCTODE_ID
							, WATERHOD_ID
							, CLEMENLM_ID
							, HICKSKS_ID
							, GREENJJ_ID
							, MALDONLL_ID
							, HENEBRMP_ID
							;v3
							, GABRIEC1_ID
							, SMITHSE3_ID
							, KULANDG_ID
							;v15
						    , LEWISMN_ID
						    , MURPHYKM_ID)*/
 
join fin
	where fin.encntr_id = outerjoin(e.encntr_id)
	  and fin.encntr_alias_type_cd = outerjoin(FIN_319_CV)
	  and fin.end_effective_dt_tm = outerjoin(cnvtdate(12312100))
 
join mrn
	where mrn.person_id = outerjoin(p.person_id)
	  and mrn.person_alias_type_cd = outerjoin(MRN_4_CV)
	  and mrn.end_effective_dt_tm = outerjoin(cnvtdate(12312100))
 
order by
	ce.parent_event_id
	, prov.person_id
 
/*select distinct into "NL:"
 
from
	tracking_checkin tc
	, tracking_item ti
	, encounter e
	, tracking_prv_reln tpr
	, prsnl prov
	, person p
	, encntr_alias fin
	, person_alias mrn
 
plan tc
	where tc.tracking_group_cd = EDTRKGRP_16370_CV
	  and tc.checkout_dt_tm between cnvtdatetime(start_date) and cnvtdatetime(end_date) ;DOS = checkout_dt_Tm
	  and tc.active_ind = 1
	  and tc.checkout_disposition_cd != LEFTWOBEINSEEN_19_CV
	  and tc.acuity_level_id in (ACLVL1_ID, ACLVL2_ID, ACLVL3_ID, ACLVL4_ID, ACLVL5_ID, ACLVLU_ID)
 
join ti
	where ti.tracking_id = tc.tracking_id
	  and ti.encntr_id > 0
 
join e
	where e.encntr_id = ti.encntr_id
	  and e.end_effective_dt_tm > cnvtdatetime(curdate, curtime3)
	  and e.active_ind = 1
 
join tpr
	where tpr.tracking_id = ti.tracking_id
	  and tpr.tracking_provider_id in (GUINSTE_ID
							, RAMIREDE_ID
							, POIRIEMP_ID
							, QURESHFA_ID
							, SARTORSC_ID
							, CLINGEJM_ID
							, MILLERJD_ID
							, PETRONKA_ID
							, WHITENJ_ID
							, SCHMIDJM_ID
							, GODAMBS_ID
							, ARZUBIMK_ID
							, KAPOORR_ID
							, BURHOPJE_ID
							, MULLANPC_ID
							, SCHACHNM_ID
							, EASONMK_ID
							, LEADERAP_ID
							, WELLERMA_ID
							, HERBERKE_ID
							, GALIOTJL_ID
							, VOKOUNKJ_ID
							, GABRIENM_ID
							, BLANCOOA_ID
							, HORNBUAC_ID
							, MILLERSF1_ID
							, SCHOCKKK_ID
							, PHILIPV_ID
							, WEAVERBJ_ID
							, ATAYO_ID
							, ASMUNDAS_ID
							;1881	MD	Carr, Daniel T.    32622595.00 <<<--- does not have a username
							, FARMERPF_ID
							, THOMASPM_ID
							, WALLENAN_ID
							, WOODAE1_ID
							, PROCTODE_ID
							, WATERHOD_ID
							, CLEMENLM_ID
							, HICKSKS_ID
							, GREENJJ_ID
							, MALDONLL_ID
							, HENEBRMP_ID
							;v3
							, GABRIEC1_ID
							, SMITHSE3_ID
							, KULANDG_ID
							;v15
						    , LEWISMN_ID
						    , MURPHYKM_ID)
	  and (tpr.unassign_dt_tm >= tc.checkout_dt_tm and tpr.updt_id != tpr.tracking_provider_id)
 
join prov
	where prov.person_id = tpr.tracking_provider_id
	  and prov.physician_ind = 1
 
join p
	where p.person_id = e.person_id
 
join fin
	where fin.encntr_id = outerjoin(e.encntr_id)
	  and fin.encntr_alias_type_cd = outerjoin(FIN_319_CV)
	  and fin.end_effective_dt_tm = outerjoin(cnvtdate(12312100))
 
join mrn
	where mrn.person_id = outerjoin(p.person_id)
	  and mrn.person_alias_type_cd = outerjoin(MRN_4_CV)
	  and mrn.end_effective_dt_tm = outerjoin(cnvtdate(12312100))
 
order by
	tpr.tracking_prv_reln_id
;	, tpr.unassign_dt_tm ;;;<<<???
*/
head report
 
	cnt = 0
 
	;Allocate memory for list with 500 records
	STAT = ALTERLIST(temp->rec, 500)
 
head ce.parent_event_id ;tpr.tracking_prv_reln_id
 
	cnt = cnt + 1
	;check for available memory in the list
	IF(MOD(cnt, 500) = 1 AND cnt > 500)
		;if needed allocate memory for 500 more records
		STAT = ALTERLIST(temp->rec, cnt + 499)
	ENDIF
 
	;loading recordset
	temp->rec[cnt].pt_encntr_id 				= e.encntr_id
	temp->rec[cnt].pt_person_id					= p.person_id
	temp->rec[cnt].pt_mrn 						= trim(mrn.alias, 3)
	temp->rec[cnt].pt_fin						= trim(fin.alias, 3)
	temp->rec[cnt].pt_name						= p.name_full_formatted
	temp->rec[cnt].pt_admit						= format(e.reg_dt_tm, "mm/dd/yyyy hh:mm;;d")
	temp->rec[cnt].checkin_dt_tm 				= format(tc.checkin_dt_tm ,"mm/dd/yyyy hh:mm;;q")
	temp->rec[cnt].checkin_dt_tm_dq8 			= tc.checkin_dt_tm
	temp->rec[cnt].depart_dt_tm					= format(tc.checkout_dt_tm ,"mm/dd/yyyy hh:mm;;q")
	temp->rec[cnt].depart_dt_tm_dq8 			= tc.checkout_dt_tm
	temp->rec[cnt].pt_med_srvc  				= uar_get_code_display(e.med_service_cd)
	temp->rec[cnt].pt_med_service_cd			= e.med_service_cd
	temp->rec[cnt].pt_type						= uar_get_code_display(e.encntr_type_cd)
	temp->rec[cnt].pt_facility					= uar_get_code_display(e.loc_facility_cd)
	temp->rec[cnt].pt_unit						= uar_get_code_display(e.loc_nurse_unit_cd)
	temp->rec[cnt].pt_room						= uar_get_code_display(e.loc_room_cd)
	temp->rec[cnt].pt_bed						= uar_get_code_display(e.loc_bed_cd)
	temp->rec[cnt].provider						= prov.name_full_formatted
	temp->rec[cnt].provider_position			= uar_get_code_display(prov.position_cd)
	temp->rec[cnt].provider_id					= prov.person_id
;	temp->rec[cnt].encntr_prsnl_r_cd			= epr.encntr_prsnl_r_cd
;	temp->rec[cnt].encntr_prsnl_r_disp			= uar_get_code_display(epr.encntr_prsnl_r_cd)
 	temp->rec[cnt].los 							= DATETIMEDIFF(tc.checkout_dt_tm, tc.checkin_dt_tm, 4)
 	temp->rec[cnt].disposition 					= uar_get_code_display(tc.checkout_disposition_cd)
 	temp->rec[cnt].tid							= ti.tracking_id
 
 	temp->rec[cnt].parent_event_id				= ce.parent_event_id
 	temp->rec[cnt].rpt 							= uar_get_code_display(ce.event_cd)
 	temp->rec[cnt].event_cd						= ce.event_cd
 
 	case(tc.acuity_level_id)
    	of ACLVL1_ID:
    		temp->rec[cnt].esi	= 1
    	of ACLVL2_ID:
    		temp->rec[cnt].esi	= 2
    	of ACLVL3_ID:
    		temp->rec[cnt].esi	= 3
    	of ACLVL4_ID:
    		temp->rec[cnt].esi	= 4
    	of ACLVL5_ID:
    		temp->rec[cnt].esi	= 5
    	of ACLVLU_ID:
    		temp->rec[cnt].esi  = 0
    endcase
 
foot report
 
	;finalizing recordset
	temp->enc_cnt = cnt
	stat = alterlist(temp->rec, cnt)
 
with nocounter
 
call echo(concat("GET CSG EMERGENCY ENCOUNTERS query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
if(size(temp->rec, 5) > 0)
/*===============================================================================================================
 
                                          	GET SIGNED KD ED POWERNOTE
 
===============================================================================================================*/
call echo(concat("GET SIGNED KD ED POWERNOTE query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
select into "NL:"
from
	(dummyt d1 with seq = value(size(temp->rec, 5)))
	, clinical_event ce
	, ce_event_prsnl cep
	, scd_story s
	, prsnl pr
 
plan d1
 
join ce
	where ce.parent_event_id = temp->rec[d1.seq].parent_event_id
	  and ce.event_end_dt_tm <= cnvtdatetime(curdate, curtime3)
	  and ce.valid_until_dt_tm = cnvtdate(12312100)
	  and ce.result_status_cd in (AUTH_8_CV
	  							, ALT_8_CV
	  							, MOD_8_CV
	  						    , TRANS_8_CV)
								
								
	  and cnvtupper(ce.event_title_text) != "DATE\TIME CORRECTION"
	  and cnvtupper(ce.event_tag) != "IN ERROR*"
	  and ce.view_level = 1
	  and ce.authentic_flag = 1
 
join cep
	where cep.event_id = ce.event_id
	  and cep.valid_until_dt_tm = cnvtdate(12312100)
	  and cep.action_type_cd = SIGN_21_CV
	  and cep.action_status_cd = COMPLETED_103_CV
	  and cep.action_prsnl_id = temp->rec[d1.seq].provider_id
 
join s
	where s.event_id = outerjoin(cep.event_id)
 
join pr
	where pr.person_id = outerjoin(s.author_id)
 
/*select into "NL:"
from
	(dummyt d1 with seq = value(size(temp->rec, 5)))
	, clinical_event ce
	, ce_event_prsnl cep
	, scd_story s
	, prsnl pr
 
plan d1
	;where (temp->rec[d1.seq].signed_dq8 = 0 or temp->rec[d1.seq].num_note = 0) ;;<<<?????
 
join ce
	where ce.encntr_id = temp->rec[d1.seq].pt_encntr_id
	  and ce.event_end_dt_tm <= cnvtdatetime(curdate, curtime3)
	  and ce.event_cd in (KDEDPOWERNOTE_72_CV, KDEDMDDICTATION_72_CV, KDEDMDDICTATIO_72_CV, KDEDMDWRITRCRD_72_CV)
	  and ce.valid_until_dt_tm = cnvtdate(12312100)
	  and ce.result_status_cd in (AUTH_8_CV
	  							, ALT_8_CV
	  							, MOD_8_CV
	  						    , TRANS_8_CV)
	  and cnvtupper(ce.event_title_text) != "DATE\TIME CORRECTION"
	  and cnvtupper(ce.event_tag) != "IN ERROR*"
	  and ce.view_level = 1
	  and ce.authentic_flag = 1
 
join cep
	where cep.event_id = ce.event_id
	  and cep.valid_until_dt_tm = cnvtdate(12312100)
	  and cep.action_type_cd = SIGN_21_CV
	  and cep.action_status_cd = COMPLETED_103_CV
	  and cep.action_prsnl_id = temp->rec[d1.seq].provider_id
 
join s
	where s.event_id = cep.event_id
 
join pr
	where pr.person_id = s.author_id
*/
order by
	d1.seq
	, cep.action_prsnl_id
	, cep.event_id ;desc
	, cep.action_dt_tm desc
 
head cep.action_prsnl_id
	null
head cep.event_id
	cnt = 0
head cep.action_dt_tm
 
 	cnt = cnt + 1
	;loading recordset
	temp->rec[d1.seq].event_id					= ce.event_id
	temp->rec[d1.seq].parent_event_id			= ce.parent_event_id
;	temp->rec[d1.seq].event_cd					= ce.event_cd
	temp->rec[d1.seq].parent_event_id			= ce.parent_event_id
	temp->rec[d1.seq].event_class_disp			= trim(uar_get_code_display(ce.event_class_cd), 3)
	temp->rec[d1.seq].result_status_disp		= concat(trim(uar_get_code_display(ce.result_status_cd), 3), " PowerNote")
;	temp->rec[d1.seq].rpt 						= uar_get_code_display(ce.event_cd)
	temp->rec[d1.seq].pt_dos					= temp->rec[d1.seq].depart_dt_tm ;= format(ce.event_end_dt_tm, "mm/dd/yyyy hh:mm;;d")
	temp->rec[d1.seq].pt_dos_dq8				= temp->rec[d1.seq].depart_dt_tm_dq8 ;ce.event_end_dt_tm
 	temp->rec[d1.seq].note_provider				= trim(pr.name_full_formatted, 3)
 	temp->rec[d1.seq].note_provider_id			= pr.person_id
 	temp->rec[d1.seq].result_title				= s.title
 	temp->rec[d1.seq].signed					= format(cep.action_dt_tm, "mm/dd/yyyy hh:mm;;d")
 	temp->rec[d1.seq].signed_dq8				= cep.action_dt_tm
 
	temp->rec[d1.seq].signed_diff_f8			= datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8))
 
	if(cep.action_dt_tm >= temp->rec[d1.seq].depart_dt_tm_dq8)
		temp->rec[d1.seq].signed_diff			= format(datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z")
	else
		temp->rec[d1.seq].signed_diff			= concat("-", format(datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z"))
	endif
 
	temp->rec[d1.seq].num_note 					= 1
 
; 	call echo(build2(temp->rec[d1.seq].pt_name, " ", temp->rec[d1.seq].provider, " ", temp->rec[d1.seq].rpt, " ", temp->rec[d1.seq].signed))
with nocounter
 
call echo(concat("GET SIGNED KD ED POWERNOTE query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                                            GET SCANNED DOCUMENTATION
 
===============================================================================================================*/
call echo(concat("GET SCANNED DOCUMENTATION query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
;this includes all notes to include downtime scanned docs
select into "NL:"
 
from
	(dummyt d1  with seq = value(size(temp->rec, 5)))
	, clinical_event ce
 
plan d1
	where temp->rec[d1.seq].num_note = 0
 
join ce
	where ce.parent_event_id = temp->rec[d1.seq].parent_event_id
	  and ce.result_status_cd in (ANT_8_CV
	  							  , AUTH_8_CV
		  						  , INPROG_8_CV
		  						  , ALT_8_CV
		  						  , MOD_8_CV
		  						  , TRANS_8_CV
		  						  , UNAUTH_8_CV)
	  and ce.event_cd = KDEDMDWRITRCRD_72_CV
	  and ce.event_end_dt_tm <= cnvtdatetime(curdate, curtime3)
	  and cnvtupper(ce.event_title_text) != "DATE\TIME CORRECTION"
	  and cnvtupper(ce.event_tag) != "IN ERROR*"
	  and ce.view_level = 1
 
order by
	d1.seq
	, ce.event_id ;desc
	, ce.event_end_dt_tm desc
 
head ce.event_id
	cnt = 0
head ce.event_end_dt_tm
	cnt = cnt + 1
 
	;loading recordset
	temp->rec[d1.seq].event_id						= ce.event_id
	temp->rec[d1.seq].event_cd						= ce.event_cd
	temp->rec[d1.seq].parent_event_id				= ce.parent_event_id
	temp->rec[d1.seq].event_class_disp				= trim(uar_get_code_display(ce.event_class_cd), 3)
	temp->rec[d1.seq].result_status_disp			= concat(trim(uar_get_code_display(ce.result_status_cd), 3), " Scanned")
	temp->rec[d1.seq].rpt 							= uar_get_code_display(ce.event_cd)
	temp->rec[d1.seq].pt_dos						= temp->rec[d1.seq].depart_dt_tm
	temp->rec[d1.seq].pt_dos_dq8					= temp->rec[d1.seq].depart_dt_tm_dq8
 	temp->rec[d1.seq].result_title					= ce.event_title_text
 	temp->rec[d1.seq].signed						= format(ce.event_end_dt_tm, "mm/dd/yyyy hh:mm;;d")
	temp->rec[d1.seq].signed_dq8					= ce.event_end_dt_tm
 
	temp->rec[d1.seq].signed_diff_f8				= datetimediff(ce.event_end_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8))
 
	if(ce.event_end_dt_tm >= temp->rec[d1.seq].depart_dt_tm_dq8)
		temp->rec[d1.seq].signed_diff				= format(datetimediff(ce.event_end_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z")
	else
		temp->rec[d1.seq].signed_diff				= concat("-", format(datetimediff(ce.event_end_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z"))
	endif
 
 	temp->rec[d1.seq].num_note 						= 1
 
with nocounter
 
call echo(concat("GET SCANNED DOCUMENTATION query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                                         GET UNSIGNED NOTES
 
===============================================================================================================*/
/*call echo(concat("GET UNSIGNED NOTES query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
select into "NL:"
from
	(dummyt d1 with seq = value(size(temp->rec, 5)))
	, clinical_event ce
	, (left join ce_event_prsnl cep on (cep.event_id = ce.event_id and cep.action_prsnl_id = temp->rec[d1.seq].provider_id))
	, (left join scd_story s on (s.event_id = cep.event_id))
	, (left join prsnl pr on (pr.person_id = s.author_id))
	, encntr_prsnl_reltn epr
 
plan d1
	where (temp->rec[d1.seq].signed_dq8 = 0 or temp->rec[d1.seq].num_note = 0) ;;<<<?????
 
join ce
	where ce.encntr_id = temp->rec[d1.seq].pt_encntr_id
	  and ce.event_end_dt_tm <= cnvtdatetime(curdate, curtime3)
	  and ce.event_cd in (KDEDPOWERNOTE_72_CV, KDEDMDDICTATION_72_CV, KDEDMDDICTATIO_72_CV, KDEDMDWRITRCRD_72_CV)
	  and ce.valid_until_dt_tm = cnvtdate(12312100)
	  and ce.result_status_cd in (AUTH_8_CV
	  							, ALT_8_CV
	  							, MOD_8_CV
	  							, ANT_8_CV
	  						    , INPROG_8_CV
	  						    , TRANS_8_CV
	  						    , UNAUTH_8_CV)
	  and cnvtupper(ce.event_title_text) != "DATE\TIME CORRECTION"
	  and cnvtupper(ce.event_tag) != "IN ERROR*"
	  and ce.view_level = 1
join cep
join s
join pr
 
join epr
	where epr.encntr_id = ce.encntr_id
	  and epr.prsnl_person_id = pr.person_id
 
order by
	d1.seq
	, cep.action_prsnl_id
	, cep.event_id ;desc
	, cep.action_dt_tm desc
 
head ce.event_id
 
	;loading recordset
	temp->rec[d1.seq].event_id					= ce.event_id
	temp->rec[d1.seq].event_cd					= ce.event_cd
	temp->rec[d1.seq].parent_event_id			= ce.parent_event_id
	temp->rec[d1.seq].event_class_disp			= trim(uar_get_code_display(ce.event_class_cd), 3)
	temp->rec[d1.seq].result_status_disp		= trim(uar_get_code_display(ce.result_status_cd), 3)
	temp->rec[d1.seq].rpt 						= uar_get_code_display(ce.event_cd)
	temp->rec[d1.seq].pt_dos					= temp->rec[d1.seq].depart_dt_tm
	temp->rec[d1.seq].pt_dos_dq8				= temp->rec[d1.seq].depart_dt_tm_dq8
	temp->rec[d1.seq].result_title				= s.title
 
 	if(cep.action_type_cd = SIGN_21_CV and cep.action_status_cd = COMPLETED_103_CV)
 
		temp->rec[d1.seq].signed_diff_f8	= datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8))
 
		if(cep.action_dt_tm >= temp->rec[d1.seq].depart_dt_tm_dq8)
			temp->rec[d1.seq].signed_diff		= format(datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z")
 
			temp->rec[d1.seq].note_provider		= trim(pr.name_full_formatted, 3)
			temp->rec[d1.seq].note_provider_id	= pr.person_id
		else
			temp->rec[d1.seq].signed_diff		= concat("-", format(datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z"))
 
			temp->rec[d1.seq].note_provider		= trim(pr.name_full_formatted, 3)
			temp->rec[d1.seq].note_provider_id	= pr.person_id
		endif
 
	endif
 
	temp->rec[d1.seq].num_note 					= 1
 
with nocounter
 
call echo(concat("GET UNSIGNED NOTES query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                                            GET RESIDENT NOTE INFO
 
===============================================================================================================*/
call echo(concat("GET RESIDENT NOTE INFO query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
select into "NL:"
from
	(dummyt d1 with seq = value(size(temp->rec, 5)))
	, clinical_event ce
	, him_event_allocation hea
	, (left join him_event_allocation hea2 on (hea2.event_id = hea.event_id
											and hea2.action_type_cd = SIGN_21_CV))
	, prsnl pr
 
plan d1
 
join ce
	where ce.parent_event_id = temp->rec[d1.seq].parent_event_id
 
join hea
	where hea.event_id = ce.event_id
	  and hea.action_type_cd = PERFORM_21_CV
join hea2
 
join pr
	where pr.person_id = hea.prsnl_id
	  and pr.position_cd = KDRESIDENT_88_CV
 
order by
	d1.seq
	, pr.person_id
	;, hea.him_event_allocation_id
	, hea.completed_dt_tm
	, hea2.completed_dt_tm
 
;head d1.seq
 
;	cnt = 0
;	stat = alterlist(temp->rec[d1.seq].provs, 5)
 
head hea.event_id
;	null
;head pr.person_id
;	null
;head hea.him_event_allocation_id
 
;	cnt = cnt + 1
;	;check for available memory in the list
;	IF(MOD(cnt, 5) = 1 AND cnt > 5)
;		;if needed allocate memory for 5 more records
;		STAT = ALTERLIST(temp->rec[d1.seq].provs, cnt + 4)
;	ENDIF
 
	temp->rec[d1.seq].resident				= trim(pr.name_full_formatted, 3)
	temp->rec[d1.seq].res_position			= uar_get_code_displaY(pr.position_cd)
 
	;case(hea.action_type_cd)
	;	of PERFORM_21_CV:
			temp->rec[d1.seq].res_performed_dt_tm 	= format(hea.completed_dt_tm, "mm/dd/yyyy hh:mm;;q")
	;	of SIGN_21_CV:
			temp->rec[d1.seq].res_sign_dt_tm		= format(hea2.completed_dt_tm, "mm/dd/yyyy hh:mm;;q")
	;endcase
	;temp->rec[d1.seq].provs[cnt].res_signed_dt_tm 		= format(cep.requested_dt_tm, "mm/dd/yyyy hh:mm;;q")
 
;foot d1.seq
 
;	STAT = ALTERLIST(temp->rec[d1.seq].provs, cnt)
 
with nocounter
 
call echo(concat("GET RESIDENT NOTE INFO query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                                      GET TRANSFER FROM RESIDENT TO PROVIDER
 
===============================================================================================================*/
call echo(concat("GET TRANSFER FROM RESIDENT TO PROVIDER query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
select into "NL:"
from
	(dummyt d1 with seq = value(size(temp->rec, 5)))
	, clinical_event ce
	, ce_event_prsnl cep
	, prsnl pr
 
plan d1
 
join ce
	where ce.parent_event_id = temp->rec[d1.seq].parent_event_id
 
join cep
	where cep.event_id = ce.event_id
	 ; and cep.action_status_cd = REQUESTED_103_CV
	  and cep.action_dt_tm < cnvtdatetime(temp->rec[d1.seq].signed_dq8)
 
join pr
	where pr.person_id = cep.action_prsnl_id
 
order by
	d1.seq
	, cep.action_prsnl_id
	, cep.event_id ;desc
	, cep.request_dt_tm desc
 
head ce.event_id
 
; 	call echo(concat("Requested Prsnl = ", trim(pr.name_full_formatted, 3), " Position = ", Uar_get_code_display(pr.position_cd)))
 					;, " Receiving Prsnl = ", trim(pr2.name_full_formatted, 3)))
 	temp->rec[d1.seq].Trans_from_Author_to_Attending_dq8 	= cep.request_dt_tm
 	temp->rec[d1.seq].Transfer_from_Author_to_Attending 	= format(cep.request_dt_tm, "mm/dd/yyyy hh:mm;;q")
 
with nocounter
 
call echo(concat("GET TRANSFER FROM RESIDENT TO PROVIDER query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                               				GET SECOND VERIFIED DT/TM
 
===============================================================================================================*/
call echo(concat("GET SECOND VERIFIED DT/TM query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
select into "NL:"
from
	(dummyt d1 with seq = value(size(temp->rec, 5)))
	, clinical_event ce
	, ce_event_prsnl cep
 
plan d1
 
join ce
	where ce.encntr_id = temp->rec[d1.seq].pt_encntr_id
	  and ce.event_end_dt_tm <= cnvtdatetime(curdate, curtime3)
	  and ce.event_cd in (KDEDPOWERNOTE_72_CV, KDEDMDDICTATION_72_CV, KDEDMDDICTATIO_72_CV, KDEDMDWRITRCRD_72_CV)
	  and ce.valid_until_dt_tm = cnvtdate(12312100)
	  and ce.result_status_cd in (AUTH_8_CV
	  							, ALT_8_CV
	  							, MOD_8_CV
	  						    , TRANS_8_CV)
	  and cnvtupper(ce.event_title_text) != "DATE\TIME CORRECTION"
	  and cnvtupper(ce.event_tag) != "IN ERROR*"
	  and ce.view_level = 1
	  and ce.authentic_flag = 1
 
join cep
	where cep.event_id = ce.event_id
	  and cep.valid_until_dt_tm = cnvtdate(12312100)
	  and cep.action_dt_tm > cnvtdatetime(temp->rec[d1.seq].signed_dq8)
	  and cep.action_type_cd = SIGN_21_CV
	  and cep.action_status_cd = COMPLETED_103_CV
	  and cep.action_prsnl_id = temp->rec[d1.seq].provider_id
 
order by
	d1.seq
	, cep.action_prsnl_id
	, cep.event_id ;desc
	, cep.action_dt_tm desc
 
head cep.event_id
	null
head cep.action_dt_tm
 
	temp->rec[d1.seq].second_signed_dq8			= cep.action_dt_tm
	temp->rec[d1.seq].second_signed				= format(cep.action_dt_tm, "mm/dd/yyyy hh:mm;;q")
 
 	temp->rec[d1.seq].second_signed_diff_f8		= datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8))
 
 	if(cep.action_dt_tm >= temp->rec[d1.seq].depart_dt_tm_dq8)
		temp->rec[d1.seq].second_signed_diff		= format(datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z")
	else
		temp->rec[d1.seq].second_signed_diff		= concat("-", format(datetimediff(cep.action_dt_tm, cnvtdatetime(temp->rec[d1.seq].depart_dt_tm_dq8)), "DD days HH hr MM m;;Z"))
	endif
 
with nocounter
 
call echo(concat("GET SECOND VERIFIED DT/TM query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                                            GET NUM NOT SIGNED/NO NOTE
 
===============================================================================================================*/
call echo(concat("GET NUM NOT SIGNED/NO NOTE query START:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
 
SELECT into "NL:"
	TEMP_ENC_CNT = temp->enc_cnt
	, REC_NUM_NOT_SIGNED = temp->rec[D1.SEQ].num_not_signed
	, NOTE_SIGNED_DIFF_F8 = temp->rec[D1.SEQ].signed_dq8
 
from
	(dummyt   d1  with seq = value(size(temp->rec, 5)))
 
plan d1
 
head d1.seq
 
 
 	;If note not signed
	if( (temp->rec[D1.SEQ].signed_dq8 = 0) and (temp->rec[d1.seq].pt_dos != " ") )
		temp->rec[d1.seq].num_not_signed = 1
	;elseif note signed
	elseif( (temp->rec[D1.SEQ].signed_dq8 != 0) and (temp->rec[d1.seq].pt_dos != " ") )
		temp->rec[d1.seq].num_signed = 1
	endif
 
WITH NOCOUNTER
 
call echo(concat("GET NUM NOT SIGNED/NO NOTE query END:  ", format(cnvtdatetime(curdate, curtime), "HH:MM:SS;;D")))
/*===============================================================================================================
 
                                        SORT BY DEPARTMENT AND CALCULATE
 
===============================================================================================================*/
call echo(concat("SORT BY DEPARTMENT AND CALCULATE query START:  ", format(curtime3, "HH:MM:SS:CC;;M")))
 
SELECT into "NL:"
 
	department = substring(1, 100, temp->rec[d1.seq].department)
	, provider = substring(1, 100, temp->rec[d1.seq].provider)
	, temp->rec[d1.seq].signed_diff_f8
 
FROM
	(DUMMYT   D1  WITH SEQ = SIZE(temp->rec, 5))
 
PLAN D1
 
order by
	department
	, provider
	, temp->rec[d1.seq].signed_diff_f8
 
head report
 
	dcnt = 0
	;Initialize record structure with 30 records
	STAT = ALTERLIST(temp2->dept, 30)
 
head department
 
	dcnt = dcnt + 1
		;check for available memory in the list
	IF(MOD(dcnt, 30) = 1 AND dcnt > 30)
		;if needed allocate memory for 30 more records
		STAT = ALTERLIST(temp2->dept, dcnt + 29)
	ENDIF
 
	temp2->dept[dcnt].department 										= "Emergency Medicine"
 
	pcnt = 0
 
	;Allocate memory for spec list with 30 records
	STAT = ALTERLIST(temp2->dept[dcnt].prov, pcnt + 30)
 
head provider
 
	pcnt = pcnt + 1
 
	;check for available memory in the list
	IF(MOD(pcnt, 30) = 1 AND pcnt > 30)
		;if needed allocate memory for 30 more records
		STAT = ALTERLIST(temp2->dept[dcnt].prov, pcnt + 29)
	ENDIF
 
	temp2->dept[dcnt].prov[pcnt].prov_id				= temp->rec[d1.seq].provider_id
	temp2->dept[dcnt].prov[pcnt].provider				= temp->rec[d1.seq].provider
 
foot provider
 
	temp2->dept[dcnt].prov[pcnt].num_encntrs			= count(temp->rec[d1.seq].pt_encntr_id)
	temp2->dept[dcnt].prov[pcnt].num_notes				= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].prov[pcnt].num_signed				= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].signed != " " and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].prov[pcnt].num_not_signed			= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].signed = " " and temp->rec[d1.seq].rpt != " ")
 
	temp2->dept[dcnt].prov[pcnt].num_within_1_day		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].signed_diff_f8 between -5.000000 and 0.999999)
	temp2->dept[dcnt].prov[pcnt].num_within_2_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].signed_diff_f8 between -5.000000 and 1.999999)
	temp2->dept[dcnt].prov[pcnt].num_within_3_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].signed_diff_f8 between -5.000000 and 2.999999)
	temp2->dept[dcnt].prov[pcnt].num_within_4_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].signed_diff_f8 >= 3.000000)
 
	temp2->dept[dcnt].prov[pcnt].percent_within_1_day	= temp2->dept[dcnt].prov[pcnt].num_within_1_day/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].percent_within_2_days	= temp2->dept[dcnt].prov[pcnt].num_within_2_days/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].percent_within_3_days	= temp2->dept[dcnt].prov[pcnt].num_within_3_days/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].percent_within_4_days	= temp2->dept[dcnt].prov[pcnt].num_within_4_days/temp2->dept[dcnt].prov[pcnt].num_notes*100
 
	temp2->dept[dcnt].prov[pcnt].percent_within_1_day_vc= concat(format(temp2->dept[dcnt].prov[pcnt].percent_within_1_day, "###.##"), "%")
 	temp2->dept[dcnt].prov[pcnt].percent_within_2_days_vc= concat(format(temp2->dept[dcnt].prov[pcnt].percent_within_2_days, "###.##"), "%")
 	temp2->dept[dcnt].prov[pcnt].percent_within_3_days_vc= concat(format(temp2->dept[dcnt].prov[pcnt].percent_within_3_days, "###.##"), "%")
 	temp2->dept[dcnt].prov[pcnt].percent_within_4_days_vc= concat(format(temp2->dept[dcnt].prov[pcnt].percent_within_4_days, "###.##"), "%")
 
 	;HIM
 	temp2->dept[dcnt].prov[pcnt].him_num_within_1_day		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].second_signed_diff_f8 between -5.000000 and 0.999999)
	temp2->dept[dcnt].prov[pcnt].him_num_within_2_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].second_signed_diff_f8 between -5.000000 and 1.999999)
	temp2->dept[dcnt].prov[pcnt].him_num_within_3_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].second_signed_diff_f8 between -5.000000 and 2.999999)
	temp2->dept[dcnt].prov[pcnt].him_num_within_4_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].second_signed_diff_f8 >= 3.000000)
 
	temp2->dept[dcnt].prov[pcnt].him_percent_within_1_day	= temp2->dept[dcnt].prov[pcnt].him_num_within_1_day/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].him_percent_within_2_days	= temp2->dept[dcnt].prov[pcnt].him_num_within_2_days/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].him_percent_within_3_days	= temp2->dept[dcnt].prov[pcnt].him_num_within_3_days/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].him_percent_within_4_days	= temp2->dept[dcnt].prov[pcnt].him_num_within_4_days/temp2->dept[dcnt].prov[pcnt].num_notes*100
 
	temp2->dept[dcnt].prov[pcnt].him_percent_within_1_day_vc= concat(format(temp2->dept[dcnt].prov[pcnt].him_percent_within_1_day, "###.##"), "%")
 	temp2->dept[dcnt].prov[pcnt].him_percent_within_2_days_vc= concat(format(temp2->dept[dcnt].prov[pcnt].him_percent_within_2_days, "###.##"), "%")
 	temp2->dept[dcnt].prov[pcnt].him_percent_within_3_days_vc= concat(format(temp2->dept[dcnt].prov[pcnt].him_percent_within_3_days, "###.##"), "%")
 	temp2->dept[dcnt].prov[pcnt].him_percent_within_4_days_vc= concat(format(temp2->dept[dcnt].prov[pcnt].him_percent_within_4_days, "###.##"), "%")
 	;
 
 	temp2->dept[dcnt].prov[pcnt].percent_not_signed		= temp2->dept[dcnt].prov[pcnt].num_not_signed/temp2->dept[dcnt].prov[pcnt].num_notes*100
 	temp2->dept[dcnt].prov[pcnt].percent_not_signed_vc	= concat(format(temp2->dept[dcnt].prov[pcnt].percent_not_signed, "###.##"), "%")
 
 	temp2->dept[dcnt].prov[pcnt].median					= median(temp->rec[d1.seq].signed_diff_f8 where temp->rec[d1.seq].rpt != " "); and temp->rec[d1.seq].signed_diff_f8 != 0.000000)
 	temp2->dept[dcnt].prov[pcnt].median_vc				= format(temp2->dept[dcnt].prov[pcnt].median, "##.####")
 
 	;HIM
 	temp2->dept[dcnt].prov[pcnt].him_median					= median(temp->rec[d1.seq].second_signed_diff_f8 where temp->rec[d1.seq].rpt != " "); and temp->rec[d1.seq].signed_diff_f8 != 0.000000)
 	temp2->dept[dcnt].prov[pcnt].him_median_vc				= format(temp2->dept[dcnt].prov[pcnt].him_median, "##.####")
 
foot department
 
 	;finalize spec list size
	stat = alterlist(temp2->dept[dcnt].prov, pcnt)
 
	temp2->dept[dcnt].num_encntrs			= count(temp->rec[d1.seq].pt_encntr_id)
	temp2->dept[dcnt].num_notes				= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].num_signed			= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].signed != " ")
	temp2->dept[dcnt].num_not_signed		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].rpt != " " and temp->rec[d1.seq].signed = " ")
 
	temp2->dept[dcnt].num_within_1_day		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].signed_diff_f8 between -5.000000 and 0.999999 and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].num_within_2_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].signed_diff_f8 between -5.000000 and 1.999999 and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].num_within_3_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].signed_diff_f8 between -5.000000 and 2.999999 and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].num_within_4_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].signed_diff_f8 >= 3.000000 and temp->rec[d1.seq].rpt != " ")
 
	temp2->dept[dcnt].percent_within_1_day	= temp2->dept[dcnt].num_within_1_day/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].percent_within_2_days	= temp2->dept[dcnt].num_within_2_days/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].percent_within_3_days	= temp2->dept[dcnt].num_within_3_days/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].percent_within_4_days	= temp2->dept[dcnt].num_within_4_days/temp2->dept[dcnt].num_notes*100
 
	temp2->dept[dcnt].percent_within_1_day_vc= concat(format(temp2->dept[dcnt].percent_within_1_day, "###.##"), "%")
 	temp2->dept[dcnt].percent_within_2_days_vc= concat(format(temp2->dept[dcnt].percent_within_2_days, "###.##"), "%")
 	temp2->dept[dcnt].percent_within_3_days_vc= concat(format(temp2->dept[dcnt].percent_within_3_days, "###.##"), "%")
 	temp2->dept[dcnt].percent_within_4_days_vc= concat(format(temp2->dept[dcnt].percent_within_4_days, "###.##"), "%")
 
 	;HIM
 	temp2->dept[dcnt].him_num_within_1_day		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].second_signed_diff_f8 between -5.000000 and 0.999999 and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].him_num_within_2_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].second_signed_diff_f8 between -5.000000 and 1.999999 and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].him_num_within_3_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].second_signed_diff_f8 between -5.000000 and 2.999999 and temp->rec[d1.seq].rpt != " ")
	temp2->dept[dcnt].him_num_within_4_days		= count(temp->rec[d1.seq].rpt where temp->rec[d1.seq].second_signed_diff_f8 >= 3.000000 and temp->rec[d1.seq].rpt != " ")
 
	temp2->dept[dcnt].him_percent_within_1_day	= temp2->dept[dcnt].him_num_within_1_day/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].him_percent_within_2_days	= temp2->dept[dcnt].him_num_within_2_days/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].him_percent_within_3_days	= temp2->dept[dcnt].him_num_within_3_days/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].him_percent_within_4_days	= temp2->dept[dcnt].him_num_within_4_days/temp2->dept[dcnt].num_notes*100
 
	temp2->dept[dcnt].him_percent_within_1_day_vc= concat(format(temp2->dept[dcnt].him_percent_within_1_day, "###.##"), "%")
 	temp2->dept[dcnt].him_percent_within_2_days_vc= concat(format(temp2->dept[dcnt].him_percent_within_2_days, "###.##"), "%")
 	temp2->dept[dcnt].him_percent_within_3_days_vc= concat(format(temp2->dept[dcnt].him_percent_within_3_days, "###.##"), "%")
 	temp2->dept[dcnt].him_percent_within_4_days_vc= concat(format(temp2->dept[dcnt].him_percent_within_4_days, "###.##"), "%")
 	;
 
 	temp2->dept[dcnt].percent_not_signed	= temp2->dept[dcnt].num_not_signed/temp2->dept[dcnt].num_notes*100
 	temp2->dept[dcnt].percent_not_signed_vc	= concat(format(temp2->dept[dcnt].percent_not_signed, "###.##"), "%")
 
foot report
 
 	;finalize dept list size
	stat = alterlist(temp2->dept, dcnt)
 
 	temp2->dept_cnt = dcnt
 
WITH NOCOUNTER
 
;sort median by provider to get median for ED
SELECT into "NL:"
	department = substring(1, 100, temp->rec[d1.seq].department)
	, prov_median = temp2->dept[d1.seq].prov[d2.seq].median
 
from
	(dummyt   d1  with seq = size(temp2->dept, 5))
	, (dummyt   d2  with seq = 1)
 
plan d1
	where maxrec(d2, size(temp2->dept[d1.seq].prov, 5))
 
join d2
 
order by
	d1.seq
	, prov_median
 
head d1.seq
null
foot d1.seq
	temp2->dept[d1.seq].median				= median(temp2->dept[d1.seq].prov[d2.seq].median where temp->rec[d1.seq].rpt != " ") ; and temp->rec[d1.seq].signed_diff_f8 != 0.000000)
 	temp2->dept[d1.seq].median_vc			= format(temp2->dept[d1.seq].median, "##.####")
 
 	;HIM
 	temp2->dept[d1.seq].him_median			= median(temp2->dept[d1.seq].prov[d2.seq].him_median where temp->rec[d1.seq].rpt != " ") ; and temp->rec[d1.seq].signed_diff_f8 != 0.000000)
 	temp2->dept[d1.seq].him_median_vc		= format(temp2->dept[d1.seq].him_median, "##.####")
 
WITH NOCOUNTER
 
call echo(concat("SORT BY DEPARTMENT AND CALCULATE query END:  ", format(curtime3, "HH:MM:SS:CC;;M")))
 
endif ;if(curqual > 0)
/*===============================================================================================================
 
                                    	   ECHORECORD FOR DEBUGGING
 
===============================================================================================================*/
 
;if(TESTENV_IND)
	if($Rpt = 2) ;detail
		call echorecord(temp)
	elseif($Rpt = 5) ;median
		call echorecord(temp2)
	endif
;endif
 
/*@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
 
                                               REPORT FORMATTING
 
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@*/
set file_path = "CCLUSERDIR:my_test.csv"
 
declare ascii_file_to_csv(file_path = vc(value)) = vc with copy
 
subroutine ascii_file_to_csv(file_path)
 
	/***** display debug messages *****/
	call echo("Executing ascii_file_to_csv subroutine...")
 
	/***** declarations *****/
	declare CNT = i4
	declare LIST_SIZE = i4
	declare IDX = i4
	declare STR_LINE = vc
	declare COL_FLG = i2
	declare COL_CNT = i4
	declare COL_TOT = i4
	declare COL_TXT = vc
	declare STR_CHAR = c1
	declare INT_CHAR = i4
 
	record column_rec
	(
		1 column [*]
			2 col_begin = i4
			2 col_head = vc
	)
 
	record datarow_rec
	(
		1 datarow [*]
			2 column [*]
				3 data = vc
	)
 
	/***** file input *****/
	free define RTL3
	define RTL3 is value(file_path)
 
	select into "nl:"
	from
		RTL3T R
 
	head report
		CNT = 0
		LIST_SIZE = 0
 
	detail
		CNT = CNT + 1
		if (CNT > LIST_SIZE)
			LIST_SIZE = LIST_SIZE + 100
			call alterlist(datarow_rec->datarow, LIST_SIZE)
		endif
		STR_LINE = R.line
 
		;establish column widths
		if (CNT = 1)
			COL_FLG = 0
			COL_CNT = 0
			COL_TOT = 0
			for (IDX = 1 to textlen(STR_LINE))
				STR_CHAR = substring(IDX, 1, STR_LINE)
				INT_CHAR = ichar(STR_CHAR)
				if (INT_CHAR between 33 and 126)
					if (COL_FLG = 0)
						;new column
						COL_CNT = COL_CNT + 1
						if (COL_CNT > COL_TOT)
							COL_TOT = COL_TOT + 10
							call alterlist(column_rec->column, COL_TOT)
						endif
						column_rec->column[COL_CNT].col_begin = IDX
						COL_TXT = STR_CHAR
					else
						COL_TXT = concat(COL_TXT, STR_CHAR)
					endif
					if (IDX = textlen(STR_LINE))
						;final column
						column_rec->column[COL_CNT].col_head = COL_TXT
					endif
					COL_FLG = 1
				else
					if (COL_FLG = 1)
						;column end
						column_rec->column[COL_CNT].col_head = COL_TXT
					endif
					COL_FLG = 0
				endif
			endfor
			COL_TOT = COL_CNT
			call alterlist(column_rec->column, COL_TOT)
		endif
 
		call echorecord(column_rec)
 
		;capture datarows
		call alterlist(datarow_rec->datarow[CNT].column, COL_TOT)
		for (IDX = 1 to COL_TOT)
			STR_BEG = column_rec->column[IDX].col_begin
			if (IDX = COL_TOT)
				STR_LEN = textlen(STR_LINE) + 1 - STR_BEG
			else
				STR_LEN = column_rec->column[IDX + 1].col_begin - STR_BEG
			endif
			datarow_rec->datarow[CNT].column[IDX].data = trim(substring(STR_BEG, STR_LEN, STR_LINE))
		endfor
 
	foot report
		LIST_SIZE = CNT
		call alterlist(datarow_rec->datarow, LIST_SIZE)
 
	with nocounter
 
	/***** output *****/
	declare DOT_LOC = i4
	declare OUTFILE = vc
 
	set DOT_LOC = findstring(".", file_path, 1, 1) - 1
	if (DOT_LOC < 0)
		set DOT_LOC = textlen(file_path)
	endif
	set OUTFILE = concat(substring(1, DOT_LOC, file_path), ".csv")
 
	select into value(OUTFILE)
	from
		(dummyt d with seq = size(datarow_rec->datarow, 5))
 
	detail
		col 0
		for (IDX = 1 to COL_TOT)
			call print(concat('"', datarow_rec->datarow[d.seq].column[IDX].data, '"'))
			if (IDX < COL_TOT)
				call print(",")
			endif
		endfor
		row +1
 
	with nocounter, maxcol = 10000, format = undefined
 
	/***** exit subroutine *****/
	call echo("Exiting ascii_file_to_csv subroutine...")
	return(OUTFILE)
 
end ;ascii_file_to_csv
/*===============================================================================================================
 
                                             OUTPUT DISTRIBUTION
 
===============================================================================================================*/
#output_distribution
 
if($rpt2 = 1) ;CSV
 
 	if(size(temp->rec, 5) > 0)
 
 		execute chkd_csg_csv_ed_prvnotetat:group1 $OUTDEV
 	else
 
	 	select into $OUTDEV
	 		NO_DATA =  "No Data Qualified"
	 	with nocounter, separator = " ", format
	 endif
endif
 
if($rpt = 2) ;Readable Detail
 
	if(size(temp->rec, 5) > 0)
 
		select distinct into $OUTDEV
			date_report_run = format(cnvtdatetime(curdate, curtime), "mm/dd/yyyy hh:mm;;q")
			, department = "Emergency Medicine"
			, attending_provider = substring(1, 150, temp->rec[d1.seq].provider)
			;, encntr_reltn_reltn = substring(1, 50, temp->rec[d1.seq].encntr_prsnl_r_disp)
			;, encntr_reltn_cd	= temp->rec[d1.seq].encntr_prsnl_r_cd
			, pt_name = substring(1, 150, temp->rec[d1.seq].pt_name)
			, pt_los = temp->rec[d1.seq].los
			, esi = temp->rec[d1.seq].esi
			, depart_disposition = substring(1, 150, temp->rec[d1.seq].disposition)
			, pt_mrn = substring(1, 30, temp->rec[d1.seq].pt_mrn)
			, pt_fin = substring(1, 30, temp->rec[d1.seq].pt_fin)
			, note = substring(1, 80, temp->rec[d1.seq].rpt)
			, result_title = substring(1, 250, temp->rec[d1.seq].result_title)
			, author = substring(1, 150, temp->rec[d1.seq].note_provider)
 
			;, num_note = temp->rec[d1.seq].num_note
			, result_status = substring(1, 80, temp->rec[d1.seq].result_status_disp)
			, Transfer_from_Author_to_Attending = substring(1, 30, temp->rec[d1.seq].Transfer_from_Author_to_Attending)
			, ed_depart_dt_tm = substring(1, 30, temp->rec[d1.seq].depart_dt_tm)
 			, resident = substring(1, 150, temp->rec[d1.seq].resident)
 			, resident_perform_dt_tm = substring(1, 30, temp->rec[d1.seq].res_performed_dt_tm)
 			, resident_sign_dt_tm = substring(1, 30, temp->rec[d1.seq].res_sign_dt_tm)
			, provider_verified_dt_tm = substring(1, 30, temp->rec[d1.seq].signed)
			, provider_second_verified_dt_tm = substring(1, 30, temp->rec[d1.seq].second_signed)
 
			, signed_diff = substring(1, 30, temp->rec[d1.seq].signed_diff)
			, signed_diff_f8 = format(temp->rec[d1.seq].signed_diff_f8, "###.####")
 
			, second_signed_diff = if(temp->rec[d1.seq].second_signed_dq8 != 0)
								   	substring(1, 30, temp->rec[d1.seq].second_signed_diff)
								   endif
			, second_signed_diff_f8 = if(temp->rec[d1.seq].second_signed_dq8 != 0)
										format(temp->rec[d1.seq].second_signed_diff_f8, "###.####")
									  endif
 
		from
			(dummyt d1 with seq = value(size(temp->rec, 5)))
 
		plan d1
 
		order by
			department
			, attending_provider
			, temp->rec[d1.seq].pt_encntr_id
			, temp->rec[d1.seq].signed
 
 		;with PCFORMAT (^"^, ^ ^ , 1), FORMAT, FORMAT = CRSTREAM, MAXREC = 10000
 		with nocounter, separator = " ", format
 
 	else
 
	 	select into $OUTDEV
	 		NO_DATA =  "No Data Qualified"
	 	with nocounter, separator = " ", format
 
 	endif
 
elseif($rpt = 3)
	call echo("REPORT TYPE = 3")
	execute chkd_csg_lo_ed_mntm_provdoc $OUTDEV
elseif($rpt = 4)
	execute chkd_csg_lo_ed_mntm_provdoc2 $OUTDEV
else
 
	select into $OUTDEV
 		Name = SUBSTRING(1, 200, TEMP2->dept[D1.SEQ].prov[D2.SEQ].provider)
 		, Month	= substring(1, 3, format(start_date, "mmm;;d")) ;substring(1, 75, temp2->date_range_disp)
 		, Number_of_Notes =	TEMP2->dept[D1.SEQ].prov[D2.SEQ].num_notes
 		, Median = TEMP2->dept[D1.SEQ].prov[D2.SEQ].median
 		, Num_Completed_in_1d = TEMP2->dept[D1.SEQ].prov[D2.SEQ].num_within_1_day
 		, 1d_Percent_Signed	= TEMP2->dept[D1.SEQ].prov[D2.SEQ].percent_within_1_day
 		, Num_completed_in_2d	= TEMP2->dept[D1.SEQ].prov[D2.SEQ].num_within_2_days
 		, 2d_Percent_Signed	= TEMP2->dept[D1.SEQ].prov[D2.SEQ].percent_within_2_days
 		, Num_Completed_in_3d = TEMP2->dept[D1.SEQ].prov[D2.SEQ].num_within_3_days
 		, Percent_completed_in_3d = TEMP2->dept[D1.SEQ].prov[D2.SEQ].percent_within_3_days
 
 	from
		(dummyt   d1  with seq = size(temp2->dept, 5))
		, (dummyt   d2  with seq = 1)
 
	plan d1
		where maxrec(d2, size(temp2->dept[d1.seq].prov, 5))
 
	join d2
 
	order by
		name
 
 	with nocounter, separator = " ", format
 
endif
 
/*===============================================================================================================
 
                                          		SUBROUTINES
 
===============================================================================================================*/
#exit_script
/*===============================================================================================================
 
                                          FREE RECORD STRUCTURES
 
===============================================================================================================*/
free record enc
 
call echo(concat("END:      ", format(curtime3, "HH:MM:SS:CC;;M")))
 
end
go
